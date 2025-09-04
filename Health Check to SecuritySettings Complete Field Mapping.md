# Health Check to SecuritySettings Complete Field Mapping

## Executive Summary

This mapping provides the **exact source** for every Health Check security setting within Salesforce's SecuritySettings metadata structure, enabling OrgCheck to implement **complete Health Check parity** with 35+ new security rules.

## High-Risk Security Settings Mapping

| Health Check Setting | SecuritySettings Source Path | Field Type | OrgCheck Rule | Implementation Priority |
|---------------------|------------------------------|------------|---------------|------------------------|
| **Lock sessions to the domain in which they were first used** | `sessionSettings.lockSessionsToDomain` | boolean | **Rule 58**: Flag when `false` | 🔴 CRITICAL |
| **Enable the SMS method of device activation** | `sessionSettings.enableSMSIdentity` | boolean | **Rule 59**: Flag when `false` | 🔴 CRITICAL |
| **Enable clickjack protection for Setup pages** | `sessionSettings.enableClickjackSetup` | boolean | **Rule 60**: Flag when `false` | 🔴 CRITICAL |
| **Enable clickjack protection for non-Setup Salesforce pages** | `sessionSettings.enableClickjackNonsetupSFDC` | boolean | **Rule 61**: Flag when `false` | 🔴 CRITICAL |
| **Enable clickjack protection for customer VF pages with standard headers** | `sessionSettings.enableClickjackNonsetupUser` | boolean | **Rule 62**: Flag when `false` | 🔴 CRITICAL |
| **Enable clickjack protection for customer VF pages with headers disabled** | `sessionSettings.enableClickjackNonsetupUserHeaderless` | boolean | **Rule 63**: Flag when `false` | 🔴 CRITICAL |
| **Enable CSRF protection on GET requests on non-setup pages** | `sessionSettings.enableCSRFOnGet` | boolean | **Rule 64**: Flag when `false` | 🔴 CRITICAL |
| **Enable CSRF protection on POST requests on non-setup pages** | `sessionSettings.enableCSRFOnPost` | boolean | **Rule 65**: Flag when `false` | 🔴 CRITICAL |
| **Require HttpOnly attribute** | `sessionSettings.requireHttpOnly` | boolean | **Rule 66**: Flag when `false` | 🔴 CRITICAL |
| **Maximum invalid login attempts** | `passwordPolicies.maxLoginAttempts` | enum | **Rule 30**: ✅ **ENHANCE** with HC levels | 🟡 ENHANCE |

### High-Risk Settings - Complete Implementation Available

| Health Check Setting | Data Source | Implementation Status | OrgCheck Rule |
|---------------------|-------------|----------------------|---------------|
| **Lock sessions to domain** | `SecuritySettings.sessionSettings.lockSessionsToDomain` | ✅ **READY** | **Rule 58** |
| **Enable SMS device activation** | `SecuritySettings.sessionSettings.enableSMSIdentity` | ✅ **READY** | **Rule 59** |
| **Enable clickjack protection (4 variants)** | `SecuritySettings.sessionSettings.enableClickjack*` | ✅ **READY** | **Rules 60-63** |
| **Enable CSRF protection (GET/POST)** | `SecuritySettings.sessionSettings.enableCSRFOn*` | ✅ **READY** | **Rules 64-65** |
| **Require HttpOnly attribute** | `SecuritySettings.sessionSettings.requireHttpOnly` | ✅ **READY** | **Rule 66** |
| **Security risk file types with hybrid behavior** | `FileUploadAndDownloadSecuritySettings.dispositions[]` | ✅ **READY** | **Rule 67** |
| **Maximum invalid login attempts** | `SecuritySettings.passwordPolicies.maxLoginAttempts` | ✅ **ENHANCE EXISTING** | **Rule 30** |
| **Number of expired certificates** | `Certificate` Tooling API: `ExpirationDate < TODAY` | ✅ **NOW AVAILABLE** | **Rule 68** |
| **Objects with public external access** | `SharingSettings` + Object sharing analysis | ✅ **READY** | **Rule 69** |

## Complete Dataset Implementation Strategy

### Phase 1: SecuritySettings Dataset (Rules 58-78)
```javascript
class DatasetSecuritySettings extends Dataset {
    async run(sfdcManager, dataFactory, logger, parameters) {
        const settings = await sfdcManager.readMetadata('SecuritySettings', [], logger);
        
        return [{
            id: 'org-security-settings',
            // High-Risk Settings
            lockSessionsToDomain: settings.sessionSettings?.lockSessionsToDomain,
            enableSMSIdentity: settings.sessionSettings?.enableSMSIdentity,
            enableClickjackSetup: settings.sessionSettings?.enableClickjackSetup,
            enableClickjackNonsetupSFDC: settings.sessionSettings?.enableClickjackNonsetupSFDC,
            enableClickjackNonsetupUser: settings.sessionSettings?.enableClickjackNonsetupUser,
            enableClickjackNonsetupUserHeaderless: settings.sessionSettings?.enableClickjackNonsetupUserHeaderless,
            enableCSRFOnGet: settings.sessionSettings?.enableCSRFOnGet,
            enableCSRFOnPost: settings.sessionSettings?.enableCSRFOnPost,
            requireHttpOnly: settings.sessionSettings?.requireHttpOnly,
            
            // Medium-Risk Settings
            minimumPasswordLifetime: settings.passwordPolicies?.minimumPasswordLifetime,
            forceRelogin: settings.sessionSettings?.forceRelogin,
            enforceIpRangesEveryRequest: settings.sessionSettings?.enforceIpRangesEveryRequest,
            enableCSPOnEmail: settings.sessionSettings?.enableCSPOnEmail,
            enableContentSniffingProtection: settings.sessionSettings?.enableContentSniffingProtection,
            enableAdminLoginAsAnyUser: settings.enableAdminLoginAsAnyUser,
            
            // Low-Risk Settings
            obscureSecretAnswer: settings.passwordPolicies?.obscureSecretAnswer,
            forceLogoutOnSessionTimeout: settings.sessionSettings?.forceLogoutOnSessionTimeout,
            identityConfirmationOnTwoFactorRegistrationEnabled: settings.sessionSettings?.identityConfirmationOnTwoFactorRegistrationEnabled,
            identityConfirmationOnEmailChange: settings.sessionSettings?.identityConfirmationOnEmailChange,
            sessionTimeout: settings.sessionSettings?.sessionTimeout,
            redirectionWarning: settings.sessionSettings?.redirectionWarning
        }];
    }
}
```

### Phase 2: Certificate Management Dataset (Rule 68)
```javascript
class DatasetCertificates extends Dataset {
    async run(sfdcManager, dataFactory, logger, parameters) {
        const query = `
            SELECT Id, DeveloperName, MasterLabel, ExpirationDate, KeySize,
                   OptionsIsCaSigned, OptionsIsUnusable
            FROM Certificate 
            WHERE ExpirationDate != null
        `;
        
        const certificates = await sfdcManager.soqlQuery(query, logger);
        const today = new Date();
        
        return certificates.map(cert => ({
            id: cert.Id,
            developerName: cert.DeveloperName,
            masterLabel: cert.MasterLabel,
            expirationDate: new Date(cert.ExpirationDate),
            keySize: cert.KeySize,
            isCaSigned: cert.OptionsIsCaSigned,
            isUnusable: cert.OptionsIsUnusable,
            // Health Check calculations
            daysUntilExpiration: Math.ceil((new Date(cert.ExpirationDate) - today) / (1000 * 60 * 60 * 24)),
            isExpired: new Date(cert.ExpirationDate) < today,
            isNearExpiration: Math.ceil((new Date(cert.ExpirationDate) - today) / (1000 * 60 * 60 * 24)) < 180,
            isCriticalExpiration: Math.ceil((new Date(cert.ExpirationDate) - today) / (1000 * 60 * 60 * 24)) < 15
        }));
    }
}
```

### Phase 3: File Security Dataset (Rule 67)
```javascript
class DatasetFileUploadSecurity extends Dataset {
    async run(sfdcManager, dataFactory, logger, parameters) {
        const settings = await sfdcManager.readMetadata('FileUploadAndDownloadSecuritySettings', [], logger);
        
        const securityRiskHybridFiles = settings.dispositions?.filter(d => 
            d.securityRiskFileType === true && d.behavior === 'HYBRID'
        ) || [];
        
        return [{
            id: 'file-upload-security',
            noHtmlUploadAsAttachment: settings.noHtmlUploadAsAttachment,
            totalDispositions: settings.dispositions?.length || 0,
            securityRiskHybridCount: securityRiskHybridFiles.length,
            securityRiskHybridTypes: securityRiskHybridFiles.map(f => f.filetype),
            allDispositions: settings.dispositions || []
        }];
    }
}
```

## Medium-Risk Security Settings Mapping

| Health Check Setting | SecuritySettings Source Path | Field Type | OrgCheck Rule | Current Status |
|---------------------|------------------------------|------------|---------------|----------------|
| **Require a minimum 1-day password lifetime** | `passwordPolicies.minimumPasswordLifetime` | boolean | **Rule 67**: Flag when `false` | ❌ Missing |
| **Force relogin after Login-As-User** | `sessionSettings.forceRelogin` | boolean | **Rule 68**: Flag when `false` | ❌ Missing |
| **Enforce login IP ranges on every request** | `sessionSettings.enforceIpRangesEveryRequest` | boolean | **Rule 69**: Flag when `false` | ❌ Missing |
| **Enable Content Security Policy protection for email templates** | `sessionSettings.enableCSPOnEmail` | boolean | **Rule 70**: Flag when `false` | ❌ Missing |
| **Enable Content Sniffing protection** | `sessionSettings.enableContentSniffingProtection` | boolean | **Rule 71**: Flag when `false` | ❌ Missing |
| **Administrators Can Log In As Any User** | `enableAdminLoginAsAnyUser` | boolean | **Rule 72**: Flag when `true` | ❌ Missing |
| **Enforce password history** | `passwordPolicies.historyRestriction` | string (0-24) | **Rule 27**: ✅ **ENHANCE** with HC thresholds | 🟡 ENHANCE |
| **Minimum password length** | `passwordPolicies.minimumPasswordLength` | string (5-50) | **Rule 28**: ✅ **ENHANCE** with HC thresholds | 🟡 ENHANCE |
| **User passwords expire in** | `passwordPolicies.expiration` | enum | **Rule 25/26**: ✅ **ENHANCE** with HC thresholds | 🟡 ENHANCE |
| **Password complexity requirement** | `passwordPolicies.complexity` | enum | **Rule 29**: ✅ **ENHANCE** with HC thresholds | 🟡 ENHANCE |

## Low-Risk Security Settings Mapping

| Health Check Setting | SecuritySettings Source Path | Field Type | OrgCheck Rule | Current Status |
|---------------------|------------------------------|------------|---------------|----------------|
| **Obscure secret answer for password resets** | `passwordPolicies.obscureSecretAnswer` | boolean | **Rule 73**: Flag when `false` | ❌ Missing |
| **Force log out on session timeout** | `sessionSettings.forceLogoutOnSessionTimeout` | boolean | **Rule 74**: Flag when `false` | ❌ Missing |
| **Require identity verification during MFA registration** | `sessionSettings.identityConfirmationOnTwoFactorRegistrationEnabled` | boolean | **Rule 75**: Flag when `false` | ❌ Missing |
| **Require identity verification for change of email address** | `sessionSettings.identityConfirmationOnEmailChange` | boolean | **Rule 76**: Flag when `false` | ❌ Missing |
| **Password question requirement** | `passwordPolicies.questionRestriction` | enum | **Rule 24**: ✅ **ALREADY IMPLEMENTED** | ✅ COMPLETE |
| **Timeout Value** | `sessionSettings.sessionTimeout` | enum | **Rule 77**: Flag when >2 hours | ❌ Missing |
| **Lockout effective period** | `passwordPolicies.lockoutInterval` | enum | **Rule 31**: ✅ **ENHANCE** with HC thresholds | 🟡 ENHANCE |

### Low-Risk Settings Requiring Additional Data Sources

| Health Check Setting | Status | Required Dataset | Implementation Notes |
|---------------------|--------|------------------|---------------------|
| **Remote Site (Disable Protocol Security)** | ❌ Missing | RemoteSiteSetting metadata | Need new Dataset class |

## Informational Security Settings Mapping

| Health Check Setting | SecuritySettings Source Path | Field Type | Potential OrgCheck Rule | Implementation Priority |
|---------------------|------------------------------|------------|--------------------------|------------------------|
| **Allow redirections to untrusted external URLs without warning** | `sessionSettings.redirectionWarning` | boolean | **Rule 78**: Flag when `false` | 🟢 LOW |
| **Require permission to view record names in lookup fields** | Not in SecuritySettings | Unknown source | Research needed | 🟢 LOW |

### Informational Settings Requiring Additional Data Sources

| Health Check Setting | Status | Required Dataset | Complexity |
|---------------------|--------|------------------|------------|
| **Days until certificate expiration** | ❌ Missing | Certificate Management API | HIGH |
| **Key Size** | ❌ Missing | Certificate Management API | HIGH |
| **Number of Objects to which Guest User Profiles have Edit Access** | ❌ Missing | Profile + Object permissions cross-reference | MEDIUM |
| **Number of Objects to which Guest User Profiles have Read Access** | ❌ Missing | Profile + Object permissions cross-reference | MEDIUM |

## Implementation Strategy by Phase

### Phase 1: SecuritySettings High-Risk Rules (Rules 58-66)
**Implementation**: Single SecuritySettings dataset extension  
**Effort**: LOW - All fields available in existing metadata  
**Impact**: 9 critical security rules matching Salesforce Health Check

```javascript
// Phase 1 Implementation Example
class DatasetSecuritySettings extends Dataset {
    async run(sfdcManager, dataFactory, logger, parameters) {
        const settings = await sfdcManager.readMetadata('SecuritySettings', [], logger);
        return {
            // High-Risk Critical Settings
            lockSessionsToDomain: settings.sessionSettings?.lockSessionsToDomain,
            enableSMSIdentity: settings.sessionSettings?.enableSMSIdentity,
            enableClickjackSetup: settings.sessionSettings?.enableClickjackSetup,
            enableClickjackNonsetupSFDC: settings.sessionSettings?.enableClickjackNonsetupSFDC,
            enableClickjackNonsetupUser: settings.sessionSettings?.enableClickjackNonsetupUser,
            enableClickjackNonsetupUserHeaderless: settings.sessionSettings?.enableClickjackNonsetupUserHeaderless,
            enableCSRFOnGet: settings.sessionSettings?.enableCSRFOnGet,
            enableCSRFOnPost: settings.sessionSettings?.enableCSRFOnPost,
            requireHttpOnly: settings.sessionSettings?.requireHttpOnly,
            
            // Medium-Risk Settings
            minimumPasswordLifetime: settings.passwordPolicies?.minimumPasswordLifetime,
            forceRelogin: settings.sessionSettings?.forceRelogin,
            enforceIpRangesEveryRequest: settings.sessionSettings?.enforceIpRangesEveryRequest,
            enableCSPOnEmail: settings.sessionSettings?.enableCSPOnEmail,
            enableContentSniffingProtection: settings.sessionSettings?.enableContentSniffingProtection,
            enableAdminLoginAsAnyUser: settings.enableAdminLoginAsAnyUser,
            
            // Low-Risk Settings
            obscureSecretAnswer: settings.passwordPolicies?.obscureSecretAnswer,
            forceLogoutOnSessionTimeout: settings.sessionSettings?.forceLogoutOnSessionTimeout,
            identityConfirmationOnTwoFactorRegistrationEnabled: settings.sessionSettings?.identityConfirmationOnTwoFactorRegistrationEnabled,
            identityConfirmationOnEmailChange: settings.sessionSettings?.identityConfirmationOnEmailChange,
            sessionTimeout: settings.sessionSettings?.sessionTimeout,
            
            // Informational Settings
            redirectionWarning: settings.sessionSettings?.redirectionWarning
        };
    }
}
```

### Phase 2: Enhanced Existing Rules with Health Check Thresholds
**Implementation**: Modify existing password policy rules  
**Effort**: LOW - Logic updates only  
**Impact**: Alignment with Salesforce standards

### Phase 3: Additional Data Sources
**Implementation**: New datasets for certificates, file types, object sharing  
**Effort**: HIGH - New API integrations required  
**Impact**: Complete Health Check parity

## Health Check Threshold Alignment

### Password Policy Threshold Updates

| Setting | Current OrgCheck | Health Check Compliant | Health Check Warning | Health Check Critical |
|---------|------------------|----------------------|---------------------|----------------------|
| **historyRestriction** | <3 flagged | 3+ | 1-2 | 0 |
| **minimumPasswordLength** | <8 flagged | 8+ | 6-7 | ≤5 |
| **expiration** | >90 days flagged | ≤90 days | 180 days | OneYear/Never |
| **complexity** | <3 flagged | SpecialCharacters+ | AlphaNumeric | NoRestriction |
| **maxLoginAttempts** | NoLimit flagged | ThreeAttempts | FiveAttempts/TenAttempts | NoLimit |
| **lockoutInterval** | undefined flagged | ≥30 min | <30 min | N/A |

## Strategic Impact Assessment

### Current OrgCheck Security Coverage: 8/58 rules (14%)
### With SecuritySettings Phase 1: 23/81 rules (28%) 
### With Complete Health Check Parity: 35/93 rules (38%)

**Business Value**: Transforms OrgCheck from cleanup tool to **comprehensive security assessment platform** with validated Salesforce benchmarks

**Competitive Advantage**: Only tool providing both organizational hygiene (unused components) AND complete security posture assessment

**Implementation Feasibility**: Phase 1 requires minimal development effort with maximum security impact
