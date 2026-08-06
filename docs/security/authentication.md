# Authentication & Role-Based Access Control

Security documentation for authentication and access control.

## Login

All users (clients and staff) access the login interface at `app.edufin.co.ke/login`.
Onboarding/registration occurs at `app.edufin.co.ke/register`.

## Role-Based Redirect

After a successful login, the system redirects users to their respective dashboards based on their assigned roles:

| Role | Redirect Destination |
|------|---------------------|
| Client | `app.edufin.co.ke/dashboard` |
| Staff (Loan Officer, KYC Verifier, System Admin, Super Admin) | `app.edufin.co.ke/admin` |

The admin panel (Filament) is served at `app.edufin.co.ke/admin` on the Laravel application domain.

*To be completed*
