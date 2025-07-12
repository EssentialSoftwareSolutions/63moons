# Angular 7 RBAC Integration Guide

This guide explains how to integrate the Role-Based Access Control (RBAC) system into your existing Angular 7 project with minimal changes.

## Overview

The RBAC system provides:
- User authentication and authorization
- Role-based access control
- Permission-based access control
- Route protection
- UI element protection
- JWT token management with refresh

## Core Components

### 1. Models (`src/app/models/user.model.ts`)
- `User` - User entity with roles
- `Role` - Role entity with permissions
- `Permission` - Permission entity
- `LoginRequest/Response` - Authentication DTOs
- `AuthState` - Authentication state interface

### 2. Services
- `AuthService` - Authentication and token management
- `RoleService` - Role management operations
- `PermissionService` - Permission checking utilities

### 3. Guards
- `AuthGuard` - Protects routes requiring authentication
- `RoleGuard` - Protects routes requiring specific roles/permissions

### 4. Directives
- `*hasPermission` - Shows/hides elements based on permissions
- `*hasRole` - Shows/hides elements based on roles

### 5. Interceptors
- `AuthInterceptor` - Adds JWT tokens to HTTP requests

## Integration Steps

### Step 1: Copy Core Files

Copy these files to your existing project:

```
src/app/
├── models/
│   └── user.model.ts
├── services/
│   ├── auth.service.ts
│   ├── role.service.ts
│   └── permission.service.ts
├── guards/
│   ├── auth.guard.ts
│   └── role.guard.ts
├── directives/
│   ├── has-permission.directive.ts
│   └── has-role.directive.ts
└── interceptors/
    └── auth.interceptor.ts
```

### Step 2: Update App Module

Add the following imports to your `app.module.ts`:

```typescript
import { ReactiveFormsModule } from '@angular/forms';
import { HttpClientModule, HTTP_INTERCEPTORS } from '@angular/common/http';

// RBAC Services
import { AuthService } from './services/auth.service';
import { RoleService } from './services/role.service';
import { PermissionService } from './services/permission.service';

// RBAC Guards
import { AuthGuard } from './guards/auth.guard';
import { RoleGuard } from './guards/role.guard';

// RBAC Directives
import { HasPermissionDirective } from './directives/has-permission.directive';
import { HasRoleDirective } from './directives/has-role.directive';

// RBAC Interceptors
import { AuthInterceptor } from './interceptors/auth.interceptor';

@NgModule({
  declarations: [
    // ... your existing components
    HasPermissionDirective,
    HasRoleDirective
  ],
  imports: [
    // ... your existing imports
    ReactiveFormsModule,
    HttpClientModule
  ],
  providers: [
    // ... your existing providers
    AuthService,
    RoleService,
    PermissionService,
    AuthGuard,
    RoleGuard,
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    }
  ]
})
export class AppModule { }
```

### Step 3: Protect Routes

Update your routing configuration:

```typescript
const routes: Routes = [
  // Public routes
  { path: 'login', component: LoginComponent },
  { path: 'unauthorized', component: UnauthorizedComponent },
  
  // Protected routes
  {
    path: 'admin',
    canActivate: [AuthGuard, RoleGuard],
    data: { roles: ['admin'] },
    children: [
      // admin routes
    ]
  },
  {
    path: 'dashboard',
    canActivate: [AuthGuard],
    component: DashboardComponent
  },
  {
    path: 'users',
    canActivate: [AuthGuard, RoleGuard],
    data: { 
      permissions: ['users:read'],
      roles: ['admin', 'manager'] 
    },
    component: UserManagementComponent
  }
];
```

### Step 4: Use Directives in Templates

Protect UI elements using the provided directives:

```html
<!-- Show element only if user has specific permission -->
<button *hasPermission="'users:delete'" (click)="deleteUser()">
  Delete User
</button>

<!-- Show element only if user has specific role -->
<div *hasRole="'admin'">
  Admin Panel
</div>

<!-- Show element if user has any of the specified roles -->
<nav *hasRole="['admin', 'manager']">
  Management Menu
</nav>

<!-- Show element if user has all specified roles -->
<div *hasRole="['admin', 'super_admin']" [requireAll]="true">
  Super Admin Features
</div>

<!-- With else template -->
<div *hasPermission="'users:write'; else noPermission">
  <button>Edit User</button>
</div>
<ng-template #noPermission>
  <p>You don't have permission to edit users</p>
</ng-template>
```

### Step 5: Use Services in Components

Inject and use the services in your components:

```typescript
import { Component } from '@angular/core';
import { AuthService } from './services/auth.service';
import { PermissionService } from './services/permission.service';

@Component({
  selector: 'app-my-component',
  templateUrl: './my-component.html'
})
export class MyComponent {
  constructor(
    private authService: AuthService,
    private permissionService: PermissionService
  ) {}

  canEditUser(): boolean {
    return this.permissionService.canAccess('users', 'write');
  }

  isAdmin(): boolean {
    return this.permissionService.hasRole('admin');
  }

  getCurrentUser() {
    return this.authService.getCurrentUser();
  }

  logout() {
    this.authService.logout();
  }
}
```

## API Integration

### Backend Requirements

Your backend should provide these endpoints:

```
POST /api/auth/login
POST /api/auth/refresh
POST /api/auth/logout
GET  /api/roles
GET  /api/permissions
```

### Login Response Format

```json
{
  "user": {
    "id": "1",
    "username": "admin",
    "email": "admin@example.com",
    "firstName": "Admin",
    "lastName": "User",
    "roles": [
      {
        "id": "1",
        "name": "admin",
        "description": "Administrator",
        "permissions": [
          {
            "id": "1",
            "name": "users:read",
            "resource": "users",
            "action": "read",
            "description": "Read users"
          }
        ]
      }
    ]
  },
  "token": "jwt-token-here",
  "refreshToken": "refresh-token-here",
  "expiresIn": 3600
}
```

## Customization

### Adding New Permissions

1. Define permissions in your backend
2. Use them in route data or directive parameters:

```typescript
// In routes
data: { permissions: ['posts:create', 'posts:publish'] }

// In templates
*hasPermission="'posts:create'"
```

### Custom Permission Logic

Extend the `PermissionService` for custom logic:

```typescript
@Injectable()
export class CustomPermissionService extends PermissionService {
  canEditPost(post: Post): boolean {
    const user = this.authService.getCurrentUser();
    return this.hasPermission('posts:edit') || 
           (post.authorId === user?.id && this.hasPermission('posts:edit_own'));
  }
}
```

## Testing

The system includes mock data for testing. To test:

1. Start the development server: `ng serve`
2. Navigate to `/login`
3. Use any username/password (mock authentication)
4. Test different user roles and permissions

## Production Considerations

1. **Security**: Always validate permissions on the backend
2. **Performance**: Consider caching user permissions
3. **Error Handling**: Implement proper error handling for auth failures
4. **Token Refresh**: The system automatically handles token refresh
5. **Logout**: Clear all stored data on logout

## Troubleshooting

### Common Issues

1. **Router outlet not working**: Ensure `RouterModule` is imported
2. **Directives not working**: Check if directives are declared in module
3. **HTTP errors**: Verify interceptor is properly configured
4. **Permission checks failing**: Ensure user data includes roles and permissions

### Debug Tips

- Use browser dev tools to inspect localStorage for tokens
- Check network tab for API calls
- Use `console.log` in services to debug permission checks
- Verify route data configuration for guards
