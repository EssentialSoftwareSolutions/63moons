# Angular 7 Role-Based Access Control (RBAC) System

A comprehensive Role-Based Access Control system built with Angular 7, designed for easy integration into existing projects with minimal changes.

## Features

- 🔐 **Authentication & Authorization**: JWT-based authentication with automatic token refresh
- 👥 **Role Management**: Hierarchical role system with flexible permission assignment
- 🛡️ **Route Protection**: Guard-based route protection with role and permission checks
- 🎯 **UI Element Protection**: Structural directives for conditional rendering based on permissions
- 🔄 **Token Management**: Automatic token refresh and secure storage
- 📱 **Responsive Design**: Mobile-friendly interface with modern styling
- 🧪 **Mock Data**: Built-in mock data for testing and development

## Quick Start

### Prerequisites

- Node.js (v10 or higher)
- Angular CLI 7.x
- npm or yarn

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd angular-rbac
```

2. Install dependencies:
```bash
npm install
```

3. Start the development server:
```bash
ng serve
```

4. Navigate to `http://localhost:4200/`

### Default Login

The system includes mock authentication. Use any username and password to log in and test different user roles.

## Project Structure

```
src/app/
├── components/           # UI Components
│   ├── login/           # Login form
│   ├── dashboard/       # Main dashboard
│   ├── user-management/ # User management interface
│   ├── role-management/ # Role management interface
│   └── unauthorized/    # Access denied page
├── services/            # Business Logic
│   ├── auth.service.ts     # Authentication service
│   ├── role.service.ts     # Role management service
│   └── permission.service.ts # Permission checking utilities
├── guards/              # Route Protection
│   ├── auth.guard.ts       # Authentication guard
│   └── role.guard.ts       # Role/permission guard
├── directives/          # UI Protection
│   ├── has-permission.directive.ts # Permission-based rendering
│   └── has-role.directive.ts       # Role-based rendering
├── interceptors/        # HTTP Interceptors
│   └── auth.interceptor.ts # JWT token interceptor
└── models/              # Data Models
    └── user.model.ts       # User, Role, Permission interfaces
```

## Core Concepts

### Users, Roles, and Permissions

- **Users** have one or more **Roles**
- **Roles** contain multiple **Permissions**
- **Permissions** define access to specific resources and actions

### Permission Format

Permissions follow the format: `resource:action`

Examples:
- `users:read` - Read user data
- `users:write` - Create/update users
- `users:delete` - Delete users
- `reports:read` - View reports
- `admin:access` - Access admin panel

## Usage Examples

### Route Protection

```typescript
const routes: Routes = [
  {
    path: 'admin',
    component: AdminComponent,
    canActivate: [AuthGuard, RoleGuard],
    data: {
      roles: ['admin'],
      permissions: ['admin:access']
    }
  }
];
```

### Template Directives

```html
<!-- Show button only if user can delete users -->
<button *hasPermission="'users:delete'" (click)="deleteUser()">
  Delete User
</button>

<!-- Show admin panel only for admins -->
<div *hasRole="'admin'">
  <h3>Admin Panel</h3>
</div>

<!-- Show element for multiple roles -->
<nav *hasRole="['admin', 'manager']">
  Management Menu
</nav>
```

### Service Usage

```typescript
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
}
```

## Integration Guide

For detailed instructions on integrating this RBAC system into your existing Angular project, see [RBAC_INTEGRATION_GUIDE.md](./RBAC_INTEGRATION_GUIDE.md).

## API Requirements

The system expects your backend to provide these endpoints:

- `POST /api/auth/login` - User authentication
- `POST /api/auth/refresh` - Token refresh
- `GET /api/roles` - Get available roles
- `GET /api/permissions` - Get available permissions

See the integration guide for detailed API specifications.

## Development

### Code Scaffolding

Run `ng generate component component-name` to generate a new component.

### Build

Run `ng build` to build the project. Use `--prod` flag for production build.

### Testing

Run `ng test` to execute unit tests via Karma.
Run `ng e2e` to execute end-to-end tests via Protractor.

## Security Considerations

⚠️ **Important**: This system provides client-side access control for UI/UX purposes only. Always implement proper server-side authorization for security.

- Validate all permissions on the backend
- Use HTTPS in production
- Implement proper CORS policies
- Regularly rotate JWT secrets
- Monitor for suspicious authentication attempts

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For questions and support, please open an issue in the repository or refer to the integration guide for common troubleshooting steps.
