The SecuritySettings metadata is **extremely interesting** for OrgCheck expansion, potentially adding 20+ new security rules. This represents a massive opportunity to enhance OrgCheck's security coverage beyond the current ProfilePasswordPolicy focus.

## High-Value Security Rule Opportunities

### 🔴 **Critical Security Gaps (Immediate Rule Candidates)**

| Setting | Current Risk | Potential OrgCheck Rule | Business Impact |
|---------|--------------|------------------------|-----------------|
| `canUsersGrantLoginAccess` | Users can grant Support access | **Rule 58**: Flag when `true` | Unauthorized admin access risk |
| `enableAdminLoginAsAnyUser` | Admins can impersonate users | **Rule 59**: Flag when `true` | Privilege escalation risk |
| `maxLoginAttempts` | Unlimited login attempts | **Rule 60**: Flag when `NoLimit` | Brute force vulnerability |
| `lockoutInterval` | No account lockout | **Rule 61**: Flag when `Forever` | Manual intervention required |
| `enforceIpRangesEveryRequest` | IP restrictions only on login | **Rule 62**: Flag when `false` | Session hijacking risk |
| `requireHttps` | HTTP connections allowed | **Rule 63**: Flag when `false` | Data transmission security |

### 🟡 **Medium Priority Security Rules**

| Category | Settings | Potential Rules | Security Rationale |
|----------|----------|-----------------|-------------------|
| **CSRF Protection** | `enableCSRFOnGet`, `enableCSRFOnPost` | Rules 64-65 | Cross-site request forgery protection |
| **Clickjack Protection** | `enableClickjack*` (4 fields) | Rules 66-69 | UI redressing attack prevention |
| **Content Security** | `enableContentSniffingProtection`, `enableCSPOnEmail` | Rules 70-71 | XSS and content injection prevention |
| **Session Security** | `lockSessionsToDomain`, `lockSessionsToIp` | Rules 72-73 | Session fixation/hijacking prevention |
| **MFA Requirements** | `enableMFADirectUILoginOptIn` | Rule 74 | Multi-factor authentication enforcement |

### 🟢 **Organizational Policy Rules**

| Policy Area | Settings | Business Value | Potential Rules |
|-------------|----------|----------------|-----------------|
| **Password Lifecycle** | `minimumPasswordLifetime`, `obscureSecretAnswer` | Complement existing password rules | Rules 75-76 |
| **Redirect Security** | `redirectBlockModeEnabled`, `redirectionWarning` | Phishing protection | Rules 77-78 |
| **Session Management** | `sessionTimeout`, `forceLogoutOnSessionTimeout` | Resource management | Rules 79-80 |
| **Modern Security** | `lockerServiceNext`, `enablePermissionsPolicy` | Modern browser security | Rules 81-82 |

## Integration Strategy with Existing OrgCheck Architecture

### **Technical Implementation Path**

```javascript
// New Dataset: DatasetSecuritySettings
class DatasetSecuritySettings extends Dataset {
    async run(sfdcManager, dataFactory, logger, parameters) {
        // Single query for org-wide security settings
        const securitySettings = await sfdcManager.readMetadata('SecuritySettings', [], logger);
        return this.transformSecuritySettings(securitySettings);
    }
    
    transformSecuritySettings(settings) {
        return {
            // Network Access
            ipRanges: settings.networkAccess?.ipRanges || [],
            enforceIpRangesEveryRequest: settings.sessionSettings?.enforceIpRangesEveryRequest,
            
            // Password Policies (org-wide baseline)
            passwordComplexity: settings.passwordPolicies?.complexity,
            passwordExpiration: settings.passwordPolicies?.expiration,
            maxLoginAttempts: settings.passwordPolicies?.maxLoginAttempts,
            lockoutInterval: settings.passwordPolicies?.lockoutInterval,
            
            // Session Security
            sessionTimeout: settings.sessionSettings?.sessionTimeout,
            lockSessionsToDomain: settings.sessionSettings?.lockSessionsToDomain,
            requireHttps: settings.sessionSettings?.requireHttps,
            
            // CSRF Protection
            enableCSRFOnGet: settings.sessionSettings?.enableCSRFOnGet,
            enableCSRFOnPost: settings.sessionSettings?.enableCSRFOnPost,
            
            // Admin Controls
            canUsersGrantLoginAccess: settings.canUsersGrantLoginAccess,
            enableAdminLoginAsAnyUser: settings.enableAdminLoginAsAnyUser,
            
            // Modern Security Features
            enableMFADirectUILoginOptIn: settings.sessionSettings?.enableMFADirectUILoginOptIn,
            enablePermissionsPolicy: settings.enablePermissionsPolicy,
            lockerServiceNext: settings.sessionSettings?.lockerServiceNext
        };
    }
}
```

### **Rule Implementation Examples**

```javascript
// SFDC_SecuritySettings.js
class SFDC_SecuritySettings extends SFDC_Base {
    computeScore(d) {
        d.score = new SFDC_Score();
        
        // Critical Security Rules
        if (d.canUsersGrantLoginAccess === true) {
            d.score.addPoints(58); // Users can grant login access to Support
        }
        
        if (d.enableAdminLoginAsAnyUser === true) {
            d.score.addPoints(59); // Admin login as any user enabled
        }
        
        if (d.maxLoginAttempts === 'NoLimit') {
            d.score.addPoints(60); // No login attempt limit
        }
        
        if (d.enforceIpRangesEveryRequest === false && d.ipRanges?.length > 0) {
            d.score.addPoints(61); // IP restrictions only enforced on login
        }
        
        // Session Security Rules
        if (d.lockSessionsToDomain === false) {
            d.score.addPoints(62); // Sessions not locked to domain
        }
        
        if (d.enableCSRFOnGet === false || d.enableCSRFOnPost === false) {
            d.score.addPoints(63); // CSRF protection disabled
        }
        
        return d.score;
    }
}
```

## Strategic Value Assessment

### **Why SecuritySettings Rules Are Critical**

1. **Organizational Baseline**: Unlike ProfilePasswordPolicy (profile-specific), SecuritySettings defines org-wide security posture
2. **Compliance Coverage**: Many settings directly relate to SOC2, ISO27001, and industry compliance requirements
3. **Attack Surface Reduction**: Rules would identify common misconfigurations that expose organizations to attacks
4. **Modern Security Adoption**: Rules can encourage adoption of newer security features (Lightning Web Security, Permissions Policy, etc.)

### **Comparison with Current OrgCheck Focus**

**Current State**: 8/58 rules (14%) focus on security, primarily password policies  
**With SecuritySettings**: Could expand to 25+ security rules (40%+ of total rules)

**Current Limitations**: 
- No org-wide security baseline assessment
- Missing modern browser security features
- No administrative control monitoring
- Limited session security coverage

**Enhanced Coverage**:
- Complete security posture assessment
- Administrative privilege monitoring  
- Modern security feature adoption tracking
- Comprehensive attack vector analysis

## Implementation Priority Ranking

### **Phase 1: Critical Security Rules (Rules 58-63)**
- Admin access controls
- Brute force protection
- Session security fundamentals
- **Estimated Impact**: Immediate security improvement for most organizations

### **Phase 2: Protection Mechanisms (Rules 64-74)**  
- CSRF/Clickjack protection
- Content security policies
- MFA enforcement
- **Estimated Impact**: Defense-in-depth enhancement

### **Phase 3: Policy Optimization (Rules 75-82)**
- Advanced password policies
- Redirect security
- Modern security features
- **Estimated Impact**: Security maturity advancement

The SecuritySettings metadata represents the single largest opportunity to enhance OrgCheck's security assessment capabilities, potentially doubling the number of security-focused rules and providing comprehensive organizational security posture evaluation.

UserInterfaceSettings presents **moderate value** for OrgCheck expansion, but with important caveats. While less critical than SecuritySettings, it offers 5-8 potential rules focused on **user productivity** and **administrative risk management**.

## Value Assessment: UserInterfaceSettings vs SecuritySettings

### **Security Impact Comparison**
- **SecuritySettings**: Direct security vulnerabilities, compliance risks, attack vectors
- **UserInterfaceSettings**: Limited security relevance, primarily productivity/governance focused

### **Rule Potential Analysis**

| Priority | Setting | Potential Rule | Category | Business Impact |
|----------|---------|----------------|----------|-----------------|
| **HIGH** | `enableCustomObjectTruncate` | **Rule 83**: Flag when `true` | Security/Data Loss | Accidental data deletion risk |
| **HIGH** | `enableDeleteFieldHistory` | **Rule 84**: Flag when `true` | Compliance/Audit | Audit trail destruction risk |
| **MEDIUM** | `enableClickjackUserPageHeaderless` | **Rule 85**: Flag when `false` | Security | Clickjack vulnerability on custom VF pages |
| **MEDIUM** | `enablePersonalCanvas` | **Rule 86**: Flag when `true` | Security/Governance | Uncontrolled app installation |
| **LOW** | `enableCollapsibleSidebar` | **Rule 87**: Flag when `true` (Classic only) | User Adoption | Indicates Salesforce Classic usage |
| **LOW** | `enableCustomSidebarOnAllPages` | **Rule 88**: Flag when `true` (Classic only) | User Adoption | Legacy UI customization |

## Critical Analysis: Limited Strategic Value

### **Strengths**
1. **Data Protection Rules**: `enableCustomObjectTruncate` and `enableDeleteFieldHistory` address real business risks
2. **Security Complement**: `enableClickjackUserPageHeaderless` extends existing clickjack protection coverage
3. **Governance Insights**: Several settings indicate organizational control philosophy

### **Limitations**
1. **Low Security Impact**: Most settings affect user experience, not security posture
2. **Productivity vs Risk**: Many "flagged" settings might actually improve user productivity
3. **Context Dependency**: Whether a setting is "good" or "bad" depends heavily on organizational needs
4. **Classic UI Bias**: Several settings only apply to legacy Salesforce Classic

## Implementation Considerations

### **Rule Logic Challenges**

```javascript
// Example: This rule logic is problematic
if (d.enableInlineEdit === false) {
    d.score.addPoints(89); // "Inline edit disabled"
}
// Problem: Disabling inline edit might be intentional for data integrity
```

### **Better Approach: Context-Aware Rules**

```javascript
// More nuanced implementation
if (d.enableCustomObjectTruncate === true) {
    d.score.addPoints(83, {
        severity: 'HIGH',
        reason: 'Users can permanently delete all custom object records',
        recommendation: 'Consider restricting to specific profiles'
    });
}

if (d.enableDeleteFieldHistory === true) {
    d.score.addPoints(84, {
        severity: 'HIGH', 
        reason: 'Audit trail can be destroyed',
        compliance: ['SOX', 'GDPR', 'HIPAA']
    });
}
```

## Strategic Recommendation

### **Implementation Priority: LOW-MEDIUM**
- Focus on SecuritySettings first (20+ high-impact rules)
- Consider UserInterfaceSettings as Phase 3 expansion
- Prioritize only the 2-3 highest-risk UI settings

### **Alternative Approach: Governance Dashboard**
Rather than scoring UI settings as "problems," consider presenting them as **organizational preferences** in a separate governance section:

```javascript
// Governance insights rather than scored rules
const governanceInsights = {
    dataProtectionSettings: {
        customObjectTruncation: d.enableCustomObjectTruncate,
        fieldHistoryDeletion: d.enableDeleteFieldHistory,
        riskLevel: calculateDataRisk(d)
    },
    userExperiencePhilosophy: {
        userControlLevel: calculateUserControl(d),
        classicVsLightning: assessUIStrategy(d)
    }
};
```

## Conclusion

UserInterfaceSettings offers **incremental value** but lacks the **immediate business impact** of SecuritySettings. The focus should remain on:

1. **Phase 1**: Complete SecuritySettings implementation (Rules 58-82)
2. **Phase 2**: Advanced ProfilePasswordPolicy rules for additional fields
3. **Phase 3**: Selective UserInterfaceSettings rules (only high-risk settings)

The most valuable UserInterfaceSettings rules would be those addressing **data destruction risks** (`enableCustomObjectTruncate`, `enableDeleteFieldHistory`) rather than user experience preferences, which may vary legitimately across organizations.

# Enhanced OrgCheck Score Rules - Complete Reference Table

## Security Rules Deep Dive: ProfilePasswordPolicy Analysis

Based on the Salesforce ProfilePasswordPolicy metadata specification, here's the enhanced analysis of the 8 security rules (24-31) that examine password policies:

### ProfilePasswordPolicy Field Mappings & Validation Logic

| Salesforce Field | Valid Values | OrgCheck Rule Impact | Security Rationale |
|------------------|--------------|---------------------|-------------------|
| `passwordQuestion` | 0 (no restrictions), 1 (cannot contain password) | Rule 24: Flags when set to 0 | Prevents users from inadvertently revealing passwords in hints |
| `passwordExpiration` | 0 (never), 30, 60, 90, 180, 365 days | Rules 25-26: Flags >90 days or 0 | Enforces regular password rotation |
| `passwordHistory` | 0-24 (previous passwords to remember) | Rule 27: Flags <3 | Prevents password reuse |
| `minimumPasswordLength` | 5-50 characters | Rule 28: Flags <8 | Ensures adequate password entropy |
| `passwordComplexity` | 0-4 (increasing complexity levels) | Rule 29: Flags <3 | Enforces character diversity |
| `maxLoginAttempts` | 0 (unlimited), 3, 5, 10 | Rule 30: Flags undefined/0 | Prevents brute force attacks |
| `lockoutInterval` | 0, 15, 30, 60 minutes | Rule 31: Flags undefined | Enforces account lockout duration |

## Complete OrgCheck Rules Reference Table

| Rule ID | Rule Name | Category | Salesforce Artifacts | Description | Technical Implementation | Security/Quality Impact |
|---------|-----------|----------|----------------------|-------------|--------------------------|-------------------------|
| **0** | Not referenced anywhere | Dependencies | Custom Labels, Flows, Lightning Pages, Lightning Aura Components, Lightning Web Components, Visualforce Components, Visualforce Pages, Static Resources | Component exists but has no incoming dependencies | **API**: Dependency API<br>**Logic**: `d?.dependencies?.hadError === false && IS_EMPTY(d.dependencies?.referenced)` | **Impact**: Storage waste, maintenance burden<br>**Action**: Safe to delete if truly unused |
| **1** | No reference anywhere for custom field | Dependencies | Custom Fields | Custom field not used in any formulas, validation rules, reports, or code | **API**: FieldDefinition + Dependency API<br>**Logic**: `d?.isCustom === true && d?.dependencies?.hadError === false && IS_EMPTY(d.dependencies?.referenced)` | **Impact**: Database bloat, UI complexity<br>**Action**: Verify business need before removal |
| **2** | No reference anywhere for apex class | Dependencies | Apex Classes | Non-test Apex class not called by any other component | **API**: ApexClass + Dependency API<br>**Logic**: `d?.isTest === false && d?.dependencies?.hadError === false && IS_EMPTY(d.dependencies?.referenced)` | **Impact**: Code debt, potential security risk<br>**Action**: Remove if no business logic needed |
| **3** | Dependency API error | Dependencies | All Metadata Types | Salesforce Dependency API failed to return reference information | **API**: Dependency API error handling<br>**Logic**: `d?.dependencies && d?.dependencies.hadError === true` | **Impact**: Cannot assess true usage<br>**Action**: Retry query or manual verification |
| **4** | API Version too old | API Version | Apex Classes, Triggers, Flows, Lightning Components, VF Pages, Email Templates | Component uses API version >3 years old, missing modern features | **Calculation**: `GET_LATEST_API_VERSION() = 3*(YEAR-2022)+53+(MONTH_OFFSET)`<br>**Logic**: `IS_OLD_APIVERSION(current, component.apiVersion)` | **Impact**: Missing features, deprecated functionality<br>**Action**: Update to latest API version |
| **5** | No assertion in Apex Test | Code Quality | Apex Test Classes | Test class contains no System.assert* statements | **Analysis**: CodeScanner counts assert statements<br>**Logic**: `d?.isTest === true && d?.nbSystemAsserts === 0` | **Impact**: False test coverage, undetected bugs<br>**Action**: Add meaningful assertions |
| **6** | No description | Documentation | Lightning Pages, Components, Workflows, Web Links, Field Sets, Validation Rules, Documents, Tabs, Email Templates, Static Resources | Component lacks documentation in description field | **Field**: Various metadata Description fields<br>**Logic**: `IS_EMPTY(d?.description)` | **Impact**: Poor maintainability, knowledge loss<br>**Action**: Add clear business purpose description |
| **7** | No description for custom component | Documentation | Custom Fields, Permission Sets, Profiles | Custom-built component lacks descriptive documentation | **Logic**: `d?.isCustom === true && IS_EMPTY(d?.description)`<br>**Field**: Custom component Description | **Impact**: Organizational knowledge gaps<br>**Action**: Document business requirements and usage |
| **8** | No explicit sharing in apex class | Code Quality | Apex Classes | Class doesn't specify sharing model (with/without/inherited sharing) | **Analysis**: SymbolTable modifiers parsing<br>**Logic**: `d?.isTest === false && d?.isClass === true && !d.specifiedSharing` | **Impact**: Unpredictable data access, security risk<br>**Action**: Add explicit sharing declaration |
| **9** | Schedulable should be scheduled | Code Quality | Apex Classes | Class implements Schedulable but isn't actively scheduled | **Detection**: SymbolTable interfaces + AsyncApexJob query<br>**Logic**: `d?.isScheduled === false && d?.isSchedulable === true` | **Impact**: Unused automation capability<br>**Action**: Schedule job or remove interface |
| **10** | Not able to compile class | Code Quality | Apex Classes | Apex class has compilation errors preventing deployment | **Status**: SymbolTable presence indicates success<br>**Logic**: `d?.needsRecompilation === true` | **Impact**: Broken functionality, deployment blocks<br>**Action**: Fix compilation errors immediately |
| **11** | No coverage for this class | Code Quality | Apex Classes | Non-test class has 0% code coverage from unit tests | **API**: ApexCodeCoverageAggregate<br>**Logic**: `d?.isTest === false && (isNaN(d.coverage) || !d.coverage)` | **Impact**: Untested code, deployment risk<br>**Action**: Create comprehensive unit tests |
| **12** | Coverage not enough | Code Quality | Apex Classes | Class has <75% code coverage (Salesforce requires 75% for deployment) | **Calculation**: `NumLinesCovered / (NumLinesCovered + NumLinesUncovered)`<br>**Logic**: `d?.coverage > 0 && d?.coverage < 0.75` | **Impact**: Deployment failure risk, untested code paths<br>**Action**: Increase test coverage to 75%+ |
| **13** | At least one testing method failed | Code Quality | Apex Test Classes | Test class contains failed test methods | **API**: ApexTestResult with Outcome != 'Pass'<br>**Logic**: `d?.isTest === true && d?.testFailedMethods?.length > 0` | **Impact**: Broken functionality, false confidence<br>**Action**: Fix failing tests before deployment |
| **14** | Apex trigger should not contain SOQL | Code Quality | Apex Triggers | Trigger contains SOQL queries (violates bulkification best practices) | **Analysis**: CodeScanner.HasSOQLFromApexCode()<br>**Logic**: `d?.hasSOQL === true` | **Impact**: Performance issues, governor limit risks<br>**Action**: Move SOQL to handler classes |
| **15** | Apex trigger should not contain DML | Code Quality | Apex Triggers | Trigger contains DML operations (violates best practices) | **Analysis**: CodeScanner.HasDMLFromApexCode()<br>**Logic**: `d?.hasDML === true` | **Impact**: Recursion risks, performance issues<br>**Action**: Move DML to service classes |
| **16** | Apex Trigger should not contain logic | Code Quality | Apex Triggers | Trigger has >5000 characters, suggesting complex business logic | **Metric**: LengthWithoutComments field<br>**Logic**: `d?.length > 5000` | **Impact**: Maintainability issues, testing complexity<br>**Action**: Extract logic to handler classes |
| **17** | No direct member for this group | Useless | Public Groups, Queues | Group exists but has no direct user or group members | **Query**: Group.GroupMembers subquery<br>**Logic**: `!d.nbDirectMembers || d?.nbDirectMembers === 0` | **Impact**: Unused sharing/access control<br>**Action**: Add members or remove group |
| **18** | Custom profile with no member | Useless | Custom Profiles | Custom profile created but no users assigned | **Query**: PermissionSetAssignment grouped by Profile<br>**Logic**: `d?.isCustom === true && d?.memberCounts === 0` | **Impact**: Configuration overhead, confusion<br>**Action**: Assign users or remove profile |
| **19** | Role with no active users | Useless | User Roles | Role has no active users in the hierarchy | **Query**: User subquery with IsActive=true<br>**Logic**: `d?.activeMembersCount === 0` | **Impact**: Unused hierarchy complexity<br>**Action**: Remove role or assign active users |
| **20** | Active user not under LEX | User Adoption | Users | Active user still using Salesforce Classic interface | **Field**: UserPreferencesLightningExperiencePreferred<br>**Logic**: `d?.onLightningExperience === false` | **Impact**: Reduced productivity, legacy risk<br>**Action**: Train users on Lightning Experience |
| **21** | Active user never logged | User Adoption | Users | Active user account has never been used for login | **Field**: User.LastLoginDate<br>**Logic**: `d?.lastLogin === null` | **Impact**: Unused licenses, security risk<br>**Action**: Deactivate or prompt first login |
| **22** | Workflow with no action | Dependencies | Workflow Rules | Workflow rule exists but has no associated actions | **Analysis**: WorkflowRule.actions + workflowTimeTriggers.actions count<br>**Logic**: `d?.hasAction === false` | **Impact**: Ineffective automation, performance overhead<br>**Action**: Add actions or deactivate rule |
| **23** | Workflow with empty time triggered list | Dependencies | Workflow Rules | Time-based workflow has triggers but no time-triggered actions | **Analysis**: workflowTimeTriggers with empty actions arrays<br>**Logic**: `d?.emptyTimeTriggers.length > 0` | **Impact**: Incomplete automation logic<br>**Action**: Configure time-based actions |
| **24** | Password policy with question containing password | Security | Profile Password Policies | Password hint can contain the actual password (**HIGH SECURITY RISK**) | **Field**: ProfilePasswordPolicy.passwordQuestion<br>**Logic**: `d?.passwordQuestion === true` (when field = '0')<br>**Salesforce Field**: `passwordQuestion` = 0 (no restrictions) | **🔴 CRITICAL**: Users may expose passwords in hints<br>**Action**: Set passwordQuestion = 1 immediately |
| **25** | Password policy with too big expiration | Security | Profile Password Policies | Password expiration >90 days reduces security | **Field**: ProfilePasswordPolicy.passwordExpiration<br>**Logic**: `d?.passwordExpiration > 90`<br>**Valid Values**: 0, 30, 60, 90, 180, 365 | **Impact**: Increased risk of password compromise<br>**Action**: Set to 90 days or less |
| **26** | Password policy with no expiration | Security | Profile Password Policies | Passwords never expire (acceptable for technical users) | **Field**: ProfilePasswordPolicy.passwordExpiration<br>**Logic**: `d?.passwordExpiration === 0`<br>**Valid Values**: 0 = never expires | **Impact**: Long-term password compromise risk<br>**Action**: Evaluate if appropriate for user type |
| **27** | Password history too small | Security | Profile Password Policies | Password history <3 allows recent password reuse | **Field**: ProfilePasswordPolicy.passwordHistory<br>**Logic**: `d?.passwordHistory < 3`<br>**Valid Range**: 0-24 previous passwords | **Impact**: Users can quickly cycle back to old passwords<br>**Action**: Set to 3+ previous passwords |
| **28** | Password minimum size too small | Security | Profile Password Policies | Minimum password length <8 characters reduces entropy | **Field**: ProfilePasswordPolicy.minimumPasswordLength<br>**Logic**: `d?.minimumPasswordLength < 8`<br>**Valid Range**: 5-50 characters | **Impact**: Easier brute force attacks<br>**Action**: Set to 8+ characters minimum |
| **29** | Password complexity too weak | Security | Profile Password Policies | Password complexity requirements insufficient | **Field**: ProfilePasswordPolicy.passwordComplexity<br>**Logic**: `d?.passwordComplexity < 3`<br>**Levels**: 0-4 (increasing complexity) | **Impact**: Weak passwords vulnerable to attacks<br>**Action**: Require level 3+ (upper, lower, number) |
| **30** | No max login attempts set | Security | Profile Password Policies | Unlimited failed login attempts enable brute force attacks | **Field**: ProfilePasswordPolicy.maxLoginAttempts<br>**Logic**: `d?.maxLoginAttempts === undefined`<br>**Valid Values**: 0 (unlimited), 3, 5, 10 | **🔴 CRITICAL**: Enables brute force attacks<br>**Action**: Set to 3-5 failed attempts |
| **31** | No lockout period set | Security | Profile Password Policies | No lockout duration after failed login attempts | **Field**: ProfilePasswordPolicy.lockoutInterval<br>**Logic**: `d?.lockoutInterval === undefined`<br>**Valid Values**: 0, 15, 30, 60 minutes | **Impact**: Immediate retry after failed attempts<br>**Action**: Set 15+ minute lockout period |
| **32** | IP Range too large | Security | Profile Login IP Restrictions | IP address range allows >100,000 addresses | **Field**: Profile.loginIpRanges converted to numeric ranges<br>**Logic**: `d?.ipRanges.filter(i => i.difference > 100000).length > 0` | **Impact**: Overly permissive network access<br>**Action**: Restrict to specific office/VPN ranges |
| **33** | Login hours too large | Security | Profile Login Hours | Login time window >20 hours per day | **Field**: Profile.loginHours per weekday<br>**Logic**: `d?.loginHours.filter(i => i.difference > 1200).length > 0` | **Impact**: Excessive access window<br>**Action**: Restrict to business hours |
| **34** | Inactive component | Useless | Validation Rules, Record Types, Triggers, Workflows, Email Templates | Component exists but is marked inactive | **Field**: Various objects' Active/IsActive field<br>**Logic**: `d?.isActive === false` | **Impact**: Configuration clutter, confusion<br>**Action**: Remove if permanently unused |
| **35** | No active version for this flow | Useless | Flows | Flow has no deployed active version | **Field**: FlowDefinition.ActiveVersionId existence<br>**Logic**: `d?.isVersionActive === false` | **Impact**: Non-functional automation<br>**Action**: Activate version or remove flow |
| **36** | Too many versions under this flow | Useless | Flows | Flow has >7 versions (maintenance complexity) | **Count**: Flow records per DefinitionId<br>**Logic**: `d?.versionsCount > 7` | **Impact**: Storage waste, version confusion<br>**Action**: Clean up old versions |
| **37** | Migrate this process builder | Useless | Process Builder (Flows) | Legacy Process Builder should migrate to Flow | **Field**: Flow.ProcessType field<br>**Logic**: `d?.currentVersionRef?.type === 'Workflow'` | **Impact**: Legacy tool limitations<br>**Action**: Migrate to modern Flow Builder |
| **38** | No description for the current version of a flow | Documentation | Flow Versions | Active flow version lacks documentation | **Field**: Active Flow version Description<br>**Logic**: `IS_EMPTY(d.currentVersionRef?.description)` | **Impact**: Maintenance difficulty<br>**Action**: Document flow purpose and logic |
| **39** | API Version too old for the current version of a flow | API Version | Flow Versions | Flow version uses API >3 years old | **Field**: Active Flow version ApiVersion<br>**Logic**: `IS_OLD_APIVERSION(current, d?.currentVersionRef?.apiVersion)` | **Impact**: Missing modern Flow features<br>**Action**: Update flow to latest API version |
| **40** | This flow is running without sharing | Security | Flows | Flow runs in system mode without sharing restrictions | **Field**: Flow.RunInMode<br>**Logic**: `d?.currentVersionRef?.runningMode === 'SystemModeWithoutSharing'` | **Impact**: Bypasses record-level security<br>**Action**: Enable sharing unless system access required |
| **41** | Too many nodes in this version | Useless | Flow Versions | Flow has >100 elements (consider Apex or subflows) | **Count**: Sum of all Flow node types<br>**Logic**: `d?.currentVersionRef?.totalNodeCount > 100` | **Impact**: Performance issues, maintenance complexity<br>**Action**: Refactor into subflows or Apex |
| **42** | Near the limit | Overuse | Org Limits | Organization approaching Salesforce limit (>80% usage) | **API**: EntityDefinition.Limits<br>**Logic**: `d?.usedPercentage >= 0.80` | **Impact**: Service degradation risk<br>**Action**: Plan capacity increases or cleanup |
| **43** | Almost all licenses are used | Overuse | Permission Set Licenses | Permission set license >80% allocated | **Calculation**: UsedLicenses/TotalLicenses<br>**Logic**: `d?.usedPercentage >= 0.80` | **Impact**: Cannot assign new features<br>**Action**: Purchase more licenses or audit usage |
| **44** | You could have licenses to free up | Useless | Permission Set Licenses | License usage count doesn't match active assignees | **Analysis**: PermissionSetAssignment distinct vs UsedCount<br>**Logic**: `d?.remainingCount > 0 && d?.distinctActiveAssigneeCount !== d?.usedCount` | **Impact**: Paying for unused licenses<br>**Action**: Audit and reclaim unused assignments |
| **45** | Role with a level >= 7 | Useless | User Roles | Role hierarchy too deep (>= 7 levels) impacts performance | **Calculation**: Recursive level from role parentage<br>**Logic**: `d?.level >= 7` | **Impact**: Query performance degradation<br>**Action**: Flatten hierarchy structure |
| **46** | Hard-coded URL suspicion in this item | Hard-coded URLs | Apex Classes, Triggers, Collaboration Groups, Fields, Home Page Components, VF Components/Pages, Web Links, Tabs, Email Templates | Contains hard-coded Salesforce domain URLs | **Analysis**: CodeScanner finds .salesforce.com, .force.* patterns<br>**Logic**: `d?.hardCodedURLs?.length > 0` | **Impact**: Broken links in sandbox/migration<br>**Action**: Use dynamic URL construction |
| **47** | Hard-coded Salesforce IDs suspicion in this item | Hard-coded IDs | Apex Classes, Triggers, Collaboration Groups, Fields, Home Page Components, VF Components/Pages, Web Links, Tabs, Email Templates | Contains hard-coded Salesforce record IDs | **Analysis**: CodeScanner finds 15/18-char ID patterns<br>**Logic**: `d?.hardCodedIDs?.length > 0` | **Impact**: Broken functionality across orgs<br>**Action**: Use SOQL queries or Custom Metadata |
| **48** | At least one successful testing method was very long | Code Quality | Apex Test Classes | Test methods taking >20 seconds to execute | **API**: ApexTestResult RunTime > 20000ms<br>**Logic**: `d?.testPassedButLongMethods?.length > 0` | **Impact**: Slow deployment, governor limits<br>**Action**: Optimize test data and queries |
| **49** | Page layout should be assigned to at least one Profile | Useless | Page Layouts | Page layout not assigned to any profile | **Query**: ProfileLayout grouped by LayoutId<br>**Logic**: `d?.profileAssignmentCount === 0` | **Impact**: Unused UI configuration<br>**Action**: Assign to profiles or remove layout |
| **50** | Hard-coded URL suspicion in this document | Hard-coded URLs | Documents | Document URL contains hard-coded Salesforce domains | **Analysis**: CodeScanner on Document.Url<br>**Logic**: `d?.isHardCodedURL === true` | **Impact**: Broken document links in other orgs<br>**Action**: Use relative URLs or dynamic references |
| **51** | Unassigned Record Type | Useless | Record Types | Record type neither default nor visible in any profile | **Analysis**: Profile.RecordTypeVisibilities<br>**Logic**: `d?.isDefault === false && d?.isAvailable === false` | **Impact**: Inaccessible record type<br>**Action**: Assign to profiles or remove |
| **52** | Email template never used | Useless | Email Templates | Email template has never been used | **Field**: EmailTemplate.LastUsedDate<br>**Logic**: `d?.lastUsedDate === null` | **Impact**: Unused automation capability<br>**Action**: Remove or implement usage |
| **53** | No description for this flow | Documentation | Flows (excluding Process Builders) | Flow lacks descriptive documentation | **Logic**: `d?.type !== 'Workflow' && IS_EMPTY(d?.description)`<br>**Field**: FlowDefinition Description | **Impact**: Maintenance challenges<br>**Action**: Document business purpose clearly |
| **54** | Hard-coded URL suspicion in this article | Hard-coded URLs | Knowledge Articles | Knowledge article contains hard-coded Salesforce domain links | **Search**: SOSL 'FIND { .salesforce.com OR .force.* }'<br>**Logic**: `d?.isHardCodedURL === true` | **Impact**: Broken help links in other orgs<br>**Action**: Use dynamic URLs in articles |
| **55** | Custom permission set with no member and only assigned to empty group(s) | Useless | Permission Sets | Custom permission set only in empty groups | **Logic**: `d?.isCustom === true && d?.isGroup === false && d?.memberCounts === 0 && d?.allIncludingGroupsAreEmpty === true` | **Impact**: Complex unused permissions<br>**Action**: Remove or assign to users |
| **56** | Custom permission set group with no member | Useless | Permission Set Groups | Custom permission set group has no members | **Logic**: `d?.isCustom === true && d?.isGroup === true && d?.memberCounts === 0` | **Impact**: Empty permission container<br>**Action**: Add permission sets or remove group |
| **57** | Custom permission set with no member and not even assigned to group | Useless | Permission Sets | Completely unassigned custom permission set | **Logic**: `d?.isCustom === true && d?.isGroup === false && d?.memberCounts === 0 && d?.permissionSetGroupIds?.length === 0` | **Impact**: Wasted configuration effort<br>**Action**: Assign to users/groups or remove |

## Rule Category Analysis with Business Impact

### 🔴 Critical Security Issues (Rules 24, 30)
- **Rule 24**: Password hints can contain passwords - **IMMEDIATE ACTION REQUIRED**
- **Rule 30**: Unlimited login attempts enable brute force - **IMMEDIATE ACTION REQUIRED**

### 🟡 High-Impact Quality Issues (Rules 10, 13, 35, 40)
- **Compilation failures** blocking deployment
- **Failed tests** creating false confidence
- **Inactive flows** breaking automation
- **System mode flows** bypassing security

### 🟢 Optimization Opportunities (Rules 0-2, 17-19, 34-37)
- **Unused components** consuming storage and causing confusion
- **Legacy processes** requiring modernization
- **Inactive configurations** adding maintenance overhead

## ProfilePasswordPolicy Complete Field Reference

### Security Configuration Matrix

| Policy Area | Field | Secure Setting | OrgCheck Rule | Business Justification |
|-------------|-------|----------------|---------------|------------------------|
| **Hint Security** | passwordQuestion | 1 (restrict) | Rule 24 | Prevents password exposure in hints |
| **Password Lifecycle** | passwordExpiration | 30-90 days | Rules 25-26 | Balances security vs usability |
| **Reuse Prevention** | passwordHistory | 3-24 | Rule 27 | Prevents cycling through recent passwords |
| **Password Strength** | minimumPasswordLength | 8-50 chars | Rule 28 | Adequate entropy against attacks |
| **Character Diversity** | passwordComplexity | 3-4 | Rule 29 | Upper/lower/number/special chars |
| **Brute Force Protection** | maxLoginAttempts | 3-10 | Rule 30 | Limits automated attack attempts |
| **Account Lockout** | lockoutInterval | 15-60 min | Rule 31 | Enforces delay after failed attempts |

### Additional ProfilePasswordPolicy Fields (Not Currently Scored)

| Field | Purpose | Recommended Setting | Future Rule Opportunity |
|-------|---------|-------------------|------------------------|
| `forgotPasswordRedirect` | Password reset link behavior | `false` for security | Could add rule for `true` values |
| `minimumPasswordLifetime` | Prevent rapid password changes | `true` for sensitive profiles | Could add rule for rapid change detection |
| `obscure` | Hide security question answers | `true` for all profiles | Could add rule for `false` values |

## Technical Architecture Summary

**Total Rules**: 58 (IDs 0-57)  
**API Integration**: 15+ Salesforce APIs (Tooling, REST, Metadata, Dependency)  
**Code Analysis**: Advanced parsing of Apex, JavaScript, HTML, and configuration  
**Security Focus**: 10 dedicated security rules covering authentication, authorization, and data protection  
**Automation Coverage**: Comprehensive analysis of Flows, Process Builder, Workflows, and Apex  
**Performance Monitoring**: Org limits, test execution times, and hierarchy complexity  

**Key Insight**: OrgCheck prioritizes identifying unused/orphaned components (24% of rules) and security misconfigurations (17% of rules), directly supporting organizational compliance and operational efficiency goals.
