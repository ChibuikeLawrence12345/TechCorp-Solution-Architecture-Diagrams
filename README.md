# TechCorp IAM Solution Architecture

## Identity as the Control Plane

### High-Level Architecture
 architecture diagrams

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
 ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ HR System │────▶│ IGA │────▶│ Identity │────▶│ Target │
│ (Workday/ │SCIM │ SailPoint/ │SCIM │ Store/ │SAML/│ Systems │
│ SuccessFac)│ │ Entra ID │ │ SSO │OIDC │ │
└─────────────┘ └─────────────┘ │ (Okta/ │
└─────────────┘
│ │ │ Entra ID) │ ┌─────────────┐
│ │ └─────────────┘ │ SaaS │
│ │ │ │ Apps │
│ │ │ └─────────────┘
│ │ ┌─────▼─────┐ ┌─────────────┐
│ │ │ Adaptive │ │ On-Prem │
│ │ │ MFA │ │ Apps │
│ │ └───────────┘ └─────────────┘
│ │
┌─────▼───────▼───┐
│ PAM Solution │
│ (CyberArk) │
└────────┬────────┘
│
┌────────▼────────┐
│ Privileged │
│ Systems │
│ (Servers, DBs, │
│ Cloud Consoles)│
└─────────────────┘
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### Component Architecture

| Layer | Component | Purpose | Protocol/Integration |
|-------|-----------|---------|---------------------|
| **Source** | Workday/SAP | Authoritative identity source | SCIM 2.0 API |
| **Governance** | SailPoint/Entra ID Governance | Lifecycle automation, certifications, role mining | Event-driven provisioning |
| **Authentication** | Okta/Entra ID | SSO, federation, policy enforcement | SAML, OIDC, LDAP |
| **Privilege** | CyberArk | JIT access, credential vaulting, session recording | RDP, SSH, API |
| **Risk** | Adaptive MFA Engine | Contextual authentication | FIDO2, Push, SMS, Biometrics |
| **Cloud** | CIEM Tool | Cloud entitlement visibility | AWS IAM, Azure AD, GCP Cloud Identity |

---

### Data Flow Diagrams

**Joiner Flow:**
1. HR system creates employee record
2. IGA receives webhook → maps role → generates entitlements
3. Identity store creates user account
4. SCIM provisioning to target apps
5. Welcome email with SSO enrollment link

**Privileged Access Flow:**
1. Admin requests elevation via PAM portal
2. Approval workflow triggered (if required)
3. JIT credentials issued, time-bound
4. Session recorded for audit
5. Credentials automatically rotated/revoked

---

### Deployment View

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
┌─────────────────────────────────────────────────────────────┐
│ Public Cloud (SaaS) │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│ │ IGA │ │ SSO │ │ PAM │ │ CIEM │ │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ │
└─────────────────────────────────────────────────────────────┘
│
│ API/SCIM
│
┌─────────────────────────────────────────────────────────────┐
│ On-Premises Data Center │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│ │ AD │ │ Legacy │ │ Main- │ │
│ │ │ │ Apps │ │ frame │ │
│ └──────────┘ └──────────┘ └──────────┘ │
└─────────────────────────────────────────────────────────────┘
│                                                         │
└─────────────────────────────────────────────────────────┘
```
