# Security Documentation

## Security Architecture

EduFin implements a multi-layered security architecture.

## Security Layers

1. **Edge Security** - Cloudflare WAF, DDoS protection
2. **Transport Security** - TLS 1.2+, mTLS for banking
3. **Application Security** - CSRF, XSS, SQLi prevention
4. **Data Security** - Encryption at rest and in transit
5. **Access Control** - RBAC, policies
6. **Audit** - Comprehensive logging

## Documentation

- [Authentication & Access Control](./authentication.md)
- [Authorization & RBAC](./authorization.md)
- [Data Protection](./data-protection.md)

See also: [TECHNICAL_ARCHITECTURE.md](../../TECHNICAL_ARCHITECTURE.md) Section 5
