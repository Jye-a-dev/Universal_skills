# Universal AuthGuards & Role-Based Access Control (RBAC) Architecture

Tài liệu chuẩn hóa kiến trúc bảo vệ Route và Endpoint cho các ứng dụng Frontend, Backend và Fullstack.

---

## 1. Nguyên Tắc Cốt Lõi (Core Principles)

1. **Phân biệt rạch ròi 401 Unauthorized vs 403 Forbidden:**
   - **401 Unauthorized:** Người dùng chưa xác thực danh tính (chưa đăng nhập, thiếu token, token hết hạn, signature không hợp lệ). Hành vi chuẩn: Chuyển hướng (Redirect) về trang Login kèm `returnUrl` (trên client) hoặc trả về header `WWW-Authenticate` (trên server).
   - **403 Forbidden:** Người dùng đã xác thực danh tính hợp lệ nhưng không đủ quyền hạn (Roles/Permissions) để truy cập tài nguyên. Hành vi chuẩn: Hiển thị trang lỗi 403 / Access Denied (trên client) hoặc trả JSON `{ "error": "FORBIDDEN", "message": "Insufficient permissions" }` (trên server). Tuyệt đối KHÔNG redirect về trang Login khi gặp 403.

2. **Defense in Depth (Bảo vệ đa tầng):**
   - Không chỉ bảo vệ ở tầng UI/Client Route Guard (vì người dùng có thể can thiệp state client hoặc gửi request trực tiếp qua curl/Postman).
   - Mọi API endpoint ở tầng Backend/Server BẮT BUỘC phải có Middleware / Interceptor / Guard kiểm tra lại danh tính và quyền hạn.

---

## 2. Ma Trận Quyền Hạn (Role-Permission Matrix)

Thiết kế quyền theo mô hình hạt mịn (Fine-grained Permissions) thay vì kiểm tra Role trực tiếp:

```typescript
export enum Role {
  SUPER_ADMIN = 'SUPER_ADMIN',
  ORG_ADMIN = 'ORG_ADMIN',
  MEMBER = 'MEMBER',
  GUEST = 'GUEST'
}

export enum Permission {
  USERS_READ = 'users:read',
  USERS_WRITE = 'users:write',
  BILLING_MANAGE = 'billing:manage',
  PROJECT_READ = 'project:read',
  PROJECT_WRITE = 'project:write'
}

export const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  [Role.SUPER_ADMIN]: Object.values(Permission),
  [Role.ORG_ADMIN]: [
    Permission.USERS_READ,
    Permission.USERS_WRITE,
    Permission.BILLING_MANAGE,
    Permission.PROJECT_READ,
    Permission.PROJECT_WRITE
  ],
  [Role.MEMBER]: [
    Permission.PROJECT_READ,
    Permission.PROJECT_WRITE
  ],
  [Role.GUEST]: [
    Permission.PROJECT_READ
  ]
};

export function hasPermission(userRole: Role, requiredPermission: Permission): boolean {
  const permissions = ROLE_PERMISSIONS[userRole] || [];
  return permissions.includes(requiredPermission);
}
```

---

## 3. Triển Khai AuthGuard Phía Client (Frontend Route Protection)

### 3.1. React / Next.js (App Router Middleware)
Áp dụng middleware tại `middleware.ts` ở root project để bảo vệ server-side trước khi render trang:

```typescript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const PROTECTED_ROUTES = ['/dashboard', '/settings', '/admin'];
const ADMIN_ROUTES = ['/admin'];

export function middleware(request: NextRequest) {
  const token = request.cookies.get('session_token')?.value;
  const userRole = request.cookies.get('user_role')?.value;
  const { pathname } = request.nextUrl;

  const isProtected = PROTECTED_ROUTES.some(route => pathname.startsWith(route));
  if (isProtected && !token) {
    const loginUrl = new URL('/login', request.url);
    loginUrl.searchParams.set('returnUrl', pathname);
    return NextResponse.redirect(loginUrl);
  }

  const isAdminRoute = ADMIN_ROUTES.some(route => pathname.startsWith(route));
  if (isAdminRoute && userRole !== 'SUPER_ADMIN') {
    return NextResponse.redirect(new URL('/403', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

### 3.2. React Client Guard Component (HOC / Layout Guard)

```tsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '@/hooks/useAuth';

interface RouteGuardProps {
  children: React.ReactNode;
  requiredPermission?: string;
}

export const AuthGuard: React.FC<RouteGuardProps> = ({ children, requiredPermission }) => {
  const { isAuthenticated, isLoading, hasPermission } = useAuth();
  const location = useLocation();

  if (isLoading) {
    return <div className="flex h-screen items-center justify-center">Loading session...</div>;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (requiredPermission && !hasPermission(requiredPermission)) {
    return <Navigate to="/403" replace />;
  }

  return <>{children}</>;
};
```

---

## 4. Triển Khai AuthGuard Phía Server (Backend API Protection)

### 4.1. Express.js / Node.js Middleware

```typescript
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

export interface AuthenticatedRequest extends Request {
  user?: {
    id: string;
    role: string;
    permissions: string[];
  };
}

export const authenticateToken = (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'UNAUTHORIZED', message: 'Access token required' });
  }

  try {
    const secret = process.env.JWT_ACCESS_SECRET!;
    const decoded = jwt.verify(token, secret) as any;
    req.user = decoded;
    next();
  } catch (err) {
    return res.status(401).json({ error: 'TOKEN_EXPIRED_OR_INVALID', message: 'Invalid token' });
  }
};

export const requirePermission = (permission: string) => {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({ error: 'UNAUTHORIZED' });
    }
    if (!req.user.permissions?.includes(permission)) {
      return res.status(403).json({ error: 'FORBIDDEN', message: `Missing required permission: ${permission}` });
    }
    next();
  };
};
```

### 4.2. NestJS Guard & Decorator

```typescript
import { Injectable, CanActivate, ExecutionContext, ForbiddenException, UnauthorizedException } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class PermissionsGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredPermissions = this.reflector.getAllAndOverride<string[]>('permissions', [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredPermissions) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    if (!user) {
      throw new UnauthorizedException();
    }

    const hasPermission = requiredPermissions.every((perm) => user.permissions?.includes(perm));
    if (!hasPermission) {
      throw new ForbiddenException('Insufficient permissions');
    }

    return true;
  }
}
```

### 4.3. Python FastAPI Dependency Guard

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import jwt
import os

security = HTTPBearer()
JWT_SECRET = os.getenv("JWT_ACCESS_SECRET", "")

def get_current_user(credentials: HTTPAuthorizationCredentials = Depends(security)):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=["HS256"])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token expired",
            headers={"WWW-Authenticate": "Bearer"},
        )
    except jwt.InvalidTokenError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication token",
            headers={"WWW-Authenticate": "Bearer"},
        )

def require_permission(required_perm: str):
    def permission_checker(current_user: dict = Depends(get_current_user)):
        user_permissions = current_user.get("permissions", [])
        if required_perm not in user_permissions:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail=f"Missing required permission: {required_perm}"
            )
        return current_user
    return permission_checker
```
