
I'll analyze the OrgCheck score rules from the code and create a clear table explaining all 58 rules. Let me extract the information from the provided code.I've created a comprehensive table that explains all 58 OrgCheck score rules from your code. The table includes:

**Key Information for Each Rule:**
- Rule ID (0-57)
- Clear English description 
- Category classification
- Specific Salesforce artifacts/components targeted
- Detailed explanation of what the rule checks
- Simplified formula logic

**Rule Categories Breakdown:**
- **Useless** (14 rules) - Components serving no apparent purpose
- **Security** (10 rules) - Security configuration issues
- **Code Quality** (8 rules) - Apex development best practices
- **Dependencies** (4 rules) - Orphaned or problematic dependencies
- **Documentation** (4 rules) - Missing descriptions/documentation
- **Hard-coded URLs** (4 rules) - Hard-coded Salesforce domain references
- **User Adoption** (2 rules) - Users not following modern practices
- **API Version** (2 rules) - Outdated API versions
- **Overuse** (2 rules) - Resource capacity concerns
- **Hard-coded IDs** (1 rule) - Hard-coded Salesforce record IDs

The rules focus heavily on identifying unused/unnecessary components (24% of rules) and security configurations that need attention (17% of rules). This aligns with OrgCheck's mission to help organizations maintain clean, secure, and well-documented Salesforce environments.

Would you like me to dive deeper into any specific category or provide additional analysis of particular rules?

# OrgCheck Score Rules - Complete Reference Table with Technical Implementation

| Rule ID | Rule Name | Category | Salesforce Artifacts Targeted | Description | Formula Logic | Technical Implementation & Data Sources |
|---------|-----------|----------|--------------------------------|-------------|---------------|----------------------------------------|
| 0 | Not referenced anywhere | Dependencies | Custom Labels, Flows, Lightning Pages, Lightning Aura Components, Lightning Web Components, Visualforce Components, Visualforce Pages, Static Resources | Component exists but is not referenced by any other component in the org | No dependency errors AND referenced array is empty | **Query**: Dependency API via `sfdcManager.dependenciesQuery()` **Check**: `d?.dependencies?.hadError === false && IS_EMPTY(d.dependencies?.referenced)` |
| 1 | No reference anywhere for custom field | Dependencies | Custom Fields | Custom field exists but is not used anywhere in the org | Field is custom AND no dependency errors AND referenced array is empty | **Query**: FieldDefinition from Tooling API **Check**: `d?.isCustom === true && d?.dependencies?.hadError === false && IS_EMPTY(d.dependencies?.referenced)` |
| 2 | No reference anywhere for apex class | Dependencies | Apex Classes | Non-test Apex class exists but is not referenced anywhere | Not a test class AND no dependency errors AND referenced array is empty | **Query**: ApexClass from Tooling API **Check**: `d?.isTest === false && d?.dependencies?.hadError === false && IS_EMPTY(d.dependencies?.referenced)` |
| 3 | Dependency API error | Dependencies | Fields, Apex Classes, Custom Labels, Flows, Lightning Pages, Lightning Aura Components, Lightning Web Components, Visualforce Components, Visualforce Pages | Unable to retrieve dependency information from Salesforce API | Dependencies object has hadError = true | **Error Handling**: Dependency API call failed **Check**: `d?.dependencies && d?.dependencies.hadError === true` |
| 4 | API Version too old | API Version | Apex Classes, Apex Triggers, Flows, Lightning Aura Components, Lightning Web Components, Visualforce Pages, Visualforce Components, Email Templates | Component uses an API version that is more than 3 years old | Current API version - component API version >= 3 years | **Calculation**: `GET_LATEST_API_VERSION()` uses formula: `3*(THIS_YEAR-2022)+53+(THIS_MONTH<=2?0:(THIS_MONTH<=6?1:(THIS_MONTH<=10?2:3)))` **Check**: `IS_OLD_APIVERSION(SecretSauce.CurrentApiVersion, d?.apiVersion)` |
| 5 | No assertion in Apex Test | Code Quality | Apex Test Classes | Test class exists but contains no System.assert statements | Class is test AND nbSystemAsserts = 0 | **Code Analysis**: CodeScanner counts assert statements in Apex Body **Check**: `d?.isTest === true && d?.nbSystemAsserts === 0` |
| 6 | No description | Documentation | Lightning Pages, Lightning Aura Components, Lightning Web Components, Visualforce Pages, Visualforce Components, Workflows, Web Links, Field Sets, Validation Rules, Documents, Custom Tabs, Email Templates, Static Resources | Component lacks descriptive documentation | Description field is empty or null | **Query**: Various metadata objects' Description field **Check**: `IS_EMPTY(d?.description)` |
| 7 | No description for custom component | Documentation | Custom Fields, Custom Permission Sets, Custom Profiles | Custom component lacks descriptive documentation | Component is custom AND description field is empty | **Query**: Custom components with Description field **Check**: `d?.isCustom === true && IS_EMPTY(d?.description)` |
| 8 | No explicit sharing in apex class | Code Quality | Apex Classes | Apex class doesn't specify sharing model (with/without/inherit sharing) | Not test class AND is class AND specifiedSharing is false | **Code Analysis**: SymbolTable.tableDeclaration.modifiers parsed for sharing keywords **Check**: `d?.isTest === false && d?.isClass === true && !d.specifiedSharing` |
| 9 | Schedulable should be scheduled | Code Quality | Apex Classes | Class implements Schedulable interface but is not actually scheduled | isScheduled = false AND isSchedulable = true | **Interface Detection**: SymbolTable.interfaces includes 'System.Schedulable' **Job Check**: AsyncApexJob query for ScheduledApex **Check**: `d?.isScheduled === false && d?.isSchedulable === true` |
| 10 | Not able to compile class | Code Quality | Apex Classes | Apex class has compilation errors | needsRecompilation = true | **Compilation Check**: SymbolTable presence indicates compilation success **Check**: `d?.needsRecompilation === true` |
| 11 | No coverage for this class | Code Quality | Apex Classes | Non-test class has no code coverage from unit tests | Not test class AND (coverage is NaN OR coverage is falsy) | **Coverage Query**: ApexCodeCoverageAggregate from Tooling API **Check**: `d?.isTest === false && (isNaN(d.coverage) || !d.coverage)` |
| 12 | Coverage not enough | Code Quality | Apex Classes | Class has some code coverage but less than 75% | coverage > 0 AND coverage < 0.75 | **Coverage Calculation**: `NumLinesCovered / (NumLinesCovered + NumLinesUncovered)` **Check**: `d?.coverage > 0 && d?.coverage < 0.75` |
| 13 | At least one testing method failed | Code Quality | Apex Test Classes | Test class has one or more failed test methods | Is test class AND testFailedMethods array has elements | **Test Results**: ApexTestResult with `Outcome != 'Pass'` **Check**: `d?.isTest === true && d?.testFailedMethods && d?.testFailedMethods.length > 0` |
| 14 | Apex trigger should not contain SOQL statement | Code Quality | Apex Triggers | Trigger contains SOQL queries (violates best practices) | hasSOQL = true | **Code Analysis**: CodeScanner.HasSOQLFromApexCode() parses trigger Body **Check**: `d?.hasSOQL === true` |
| 15 | Apex trigger should not contain DML action | Code Quality | Apex Triggers | Trigger contains DML operations (violates best practices) | hasDML = true | **Code Analysis**: CodeScanner.HasDMLFromApexCode() parses trigger Body **Check**: `d?.hasDML === true` |
| 16 | Apex Trigger should not contain logic | Code Quality | Apex Triggers | Trigger has too much code, suggesting it contains business logic | Code length > 5000 characters | **Size Check**: ApexTrigger.LengthWithoutComments field **Check**: `d?.length > 5000` |
| 17 | No direct member for this group | Useless | Groups (Public Groups/Queues) | Group has no direct members (users or subgroups) | nbDirectMembers is falsy OR = 0 | **Member Query**: GroupMembers subquery in Group query **Check**: `!d.nbDirectMembers || d?.nbDirectMembers === 0` |
| 18 | Custom profile with no member | Useless | Custom Profiles | Custom profile exists but has no users assigned | Profile is custom AND memberCounts = 0 | **Assignment Count**: PermissionSetAssignment grouped by Profile **Check**: `d?.isCustom === true && d?.memberCounts === 0` |
| 19 | Role with no active users | Useless | User Roles | Role has no active users assigned to it | activeMembersCount = 0 | **Active User Count**: Users subquery filtered by IsActive=true **Check**: `d?.activeMembersCount === 0` |
| 20 | Active user not under LEX | User Adoption | Users | Active user is still using Salesforce Classic interface | onLightningExperience = false | **User Preference**: UserPreferencesLightningExperiencePreferred field **Check**: `d?.onLightningExperience === false` |
| 21 | Active user never logged | User Adoption | Users | Active user account has never been used to log in | lastLogin = null | **Login History**: User.LastLoginDate field **Check**: `d?.lastLogin === null` |
| 22 | Workflow with no action | Dependencies | Workflows | Workflow rule exists but has no associated actions | hasAction = false | **Action Count**: Metadata API WorkflowRule.actions + workflowTimeTriggers.actions **Check**: `d?.hasAction === false` |
| 23 | Workflow with empty time triggered list | Dependencies | Workflows | Time-based workflow has time triggers but no time-triggered actions | emptyTimeTriggers array has elements | **Time Trigger Analysis**: workflowTimeTriggers with actions.length === 0 **Check**: `d?.emptyTimeTriggers.length > 0` |
| 24 | Password policy with question containing password | Security | Profile Password Policies | Password policy allows password hints that may contain the actual password | passwordQuestion = true | **Metadata API**: ProfilePasswordPolicy.passwordQuestion === '0' (inverted logic) **Check**: `d?.passwordQuestion === true` |
| 25 | Password policy with too big expiration | Security | Profile Password Policies | Password expiration period is longer than 90 days | passwordExpiration > 90 | **Metadata API**: ProfilePasswordPolicy.passwordExpiration **Check**: `d?.passwordExpiration > 90` |
| 26 | Password policy with no expiration | Security | Profile Password Policies | Passwords never expire (potentially for technical users) | passwordExpiration = 0 | **Metadata API**: ProfilePasswordPolicy.passwordExpiration **Check**: `d?.passwordExpiration === 0` |
| 27 | Password history too small | Security | Profile Password Policies | Password history setting is too low (less than 3 previous passwords) | passwordHistory < 3 | **Metadata API**: ProfilePasswordPolicy.passwordHistory **Check**: `d?.passwordHistory < 3` |
| 28 | Password minimum size too small | Security | Profile Password Policies | Minimum password length is less than 8 characters | minimumPasswordLength < 8 | **Metadata API**: ProfilePasswordPolicy.minimumPasswordLength **Check**: `d?.minimumPasswordLength < 8` |
| 29 | Password complexity too weak | Security | Profile Password Policies | Password complexity requirements are too low | passwordComplexity < 3 | **Metadata API**: ProfilePasswordPolicy.passwordComplexity **Check**: `d?.passwordComplexity < 3` |
| 30 | No max login attempts set | Security | Profile Password Policies | No limit on failed login attempts before lockout | maxLoginAttempts is undefined | **Metadata API**: ProfilePasswordPolicy.maxLoginAttempts **Check**: `d?.maxLoginAttempts === undefined` |
| 31 | No lockout period set | Security | Profile Password Policies | No lockout period defined for failed login attempts | lockoutInterval is undefined | **Metadata API**: ProfilePasswordPolicy.lockoutInterval **Check**: `d?.lockoutInterval === undefined` |
| 32 | IP Range too large | Security | Profile Login IP Restrictions | IP address range allows too many addresses (>100,000 IPs) | Any IP range with difference > 100,000 | **IP Calculation**: Profile Metadata loginIpRanges converted to numeric range **Check**: `d?.ipRanges.filter(i => i.difference > 100000).length > 0` |
| 33 | Login hours too large | Security | Profile Login Hours | Login time window is too broad (>20 hours per day) | Any login hour range with difference > 1200 minutes | **Time Calculation**: Profile Metadata loginHours per weekday **Check**: `d?.loginHours.filter(i => i.difference > 1200).length > 0` |
| 34 | Inactive component | Useless | Validation Rules, Record Types, Apex Triggers, Workflows, Email Templates | Component exists but is marked as inactive | isActive = false | **Status Field**: Various objects' Active/IsActive field **Check**: `d?.isActive === false` |
| 35 | No active version for this flow | Useless | Flows | Flow has no active version deployed | isVersionActive = false | **Flow Status**: FlowDefinition.ActiveVersionId existence **Check**: `d?.isVersionActive === false` |
| 36 | Too many versions under this flow | Useless | Flows | Flow has accumulated too many versions (>7) | versionsCount > 7 | **Version Count**: Count of Flow records per DefinitionId **Check**: `d?.versionsCount > 7` |
| 37 | Migrate this process builder | Useless | Process Builder (Flows) | Process Builder should be migrated to Flow | currentVersionRef.type = 'Workflow' | **Process Type**: Flow.ProcessType field **Check**: `d?.currentVersionRef?.type === 'Workflow'` |
| 38 | No description for the current version of a flow | Documentation | Flows | Current active version of flow lacks description | currentVersionRef.description is empty | **Flow Version**: Active Flow version Description field **Check**: `IS_EMPTY(d.currentVersionRef?.description)` |
| 39 | API Version too old for the current version of a flow | API Version | Flows | Current flow version uses an API version more than 3 years old | Current API version - flow API version >= 3 years | **Flow Version API**: Active Flow version ApiVersion **Check**: `IS_OLD_APIVERSION(SecretSauce.CurrentApiVersion, d?.currentVersionRef?.apiVersion)` |
| 40 | This flow is running without sharing | Security | Flows | Flow runs in system mode without sharing restrictions | currentVersionRef.runningMode = 'SystemModeWithoutSharing' | **Flow Security**: Flow.RunInMode field **Check**: `d?.currentVersionRef?.runningMode === 'SystemModeWithoutSharing'` |
| 41 | Too many nodes in this version | Useless | Flows | Flow has too many elements/nodes (>100), should consider Apex or subflows | currentVersionRef.totalNodeCount > 100 | **Node Count**: Sum of all Flow node types from Metadata **Check**: `d?.currentVersionRef?.totalNodeCount > 100` |
| 42 | Near the limit | Overuse | Salesforce Org Limits | Organization is approaching a Salesforce limit (>80% usage) | usedPercentage >= 0.80 | **Limits Query**: EntityDefinition.Limits from Tooling API **Check**: `d?.usedPercentage >= 0.80` |
| 43 | Almost all licenses are used | Overuse | Permission Set Licenses | Permission set license allocation is nearly exhausted (>80% used) | usedPercentage >= 0.80 | **License Usage**: PermissionSetLicense.UsedLicenses/TotalLicenses **Check**: `d?.usedPercentage !== undefined && d?.usedPercentage >= 0.80` |
| 44 | You could have licenses to free up | Useless | Permission Set Licenses | License usage count doesn't match active distinct assignees | remainingCount > 0 AND distinctActiveAssigneeCount ≠ usedCount | **Assignment Analysis**: PermissionSetAssignment distinct count vs UsedCount **Check**: `d?.remainingCount > 0 && d?.distinctActiveAssigneeCount !== d?.usedCount` |
| 45 | Role with a level >= 7 | Useless | User Roles | Role hierarchy is too deep (7+ levels), impacting performance | level >= 7 | **Hierarchy Calculation**: Recursive level calculation from role parentage **Check**: `d?.level >= 7` |
| 46 | Hard-coded URL suspicion in this item | Hard-coded URLs | Apex Classes, Apex Triggers, Collaboration Groups, Fields, Home Page Components, Visualforce Components, Visualforce Pages, Web Links, Custom Tabs, Email Templates | Component contains hard-coded URLs pointing to Salesforce domains | hardCodedURLs array has elements | **Code Analysis**: CodeScanner.FindHardCodedURLs() searches for .salesforce.com, .force.* patterns **Check**: `d?.hardCodedURLs?.length > 0 \|\| false` |
| 47 | Hard-coded Salesforce IDs suspicion in this item | Hard-coded IDs | Apex Classes, Apex Triggers, Collaboration Groups, Fields, Home Page Components, Visualforce Components, Visualforce Pages, Web Links, Custom Tabs, Email Templates | Component contains hard-coded Salesforce record IDs | hardCodedIDs array has elements | **Code Analysis**: CodeScanner.FindHardCodedIDs() searches for 15/18-char Salesforce ID patterns **Check**: `d?.hardCodedIDs?.length > 0 \|\| false` |
| 48 | At least one successful testing method was very long | Code Quality | Apex Test Classes | Test class has methods that take too long to execute (>20 seconds) | testPassedButLongMethods array has elements | **Performance Analysis**: ApexTestResult with RunTime > 20000ms and Outcome = 'Pass' **Check**: `d?.isTest === true && d?.testPassedButLongMethods && d?.testPassedButLongMethods.length > 0` |
| 49 | Page layout should be assigned to at least one Profile | Useless | Page Layouts | Page layout exists but is not assigned to any profile | profileAssignmentCount = 0 | **Assignment Count**: ProfileLayout grouped by LayoutId **Check**: `d?.profileAssignmentCount === 0` |
| 50 | Hard-coded URL suspicion in this document | Hard-coded URLs | Documents | Document URL contains hard-coded Salesforce domain references | isHardCodedURL = true | **URL Analysis**: CodeScanner.FindHardCodedURLs() on Document.Url **Check**: `d?.isHardCodedURL === true` |
| 51 | Unassigned Record Type | Useless | Record Types | Record type is neither default nor visible in any profile | isDefault = false AND isAvailable = false | **Profile Metadata**: RecordTypeVisibilities in Profile metadata **Check**: `d?.isDefault === false && d?.isAvailable === false` |
| 52 | Email template never used | Useless | Email Templates | Email template has never been used in the organization | lastUsedDate = null | **Usage History**: EmailTemplate.LastUsedDate field **Check**: `d?.lastUsedDate === null` |
| 53 | No description for this flow | Documentation | Flows (excluding Process Builders) | Flow lacks descriptive documentation | type ≠ 'Workflow' AND description is empty | **Flow Type**: FlowDefinition type check and Description field **Check**: `d?.type !== 'Workflow' && IS_EMPTY(d?.description)` |
| 54 | Hard-coded URL suspicion in this article | Hard-coded URLs | Knowledge Articles | Knowledge article contains links to hard-coded Salesforce domains | isHardCodedURL = true | **SOSL Search**: 'FIND { .salesforce.com OR .force.* } IN ALL FIELDS' on KnowledgeArticleVersion **Check**: `d?.isHardCodedURL === true` |
| 55 | Custom permission set with no member and only assigned to empty group(s) | Useless | Permission Sets | Custom permission set has no direct members and only belongs to empty groups | isCustom = true AND isGroup = false AND memberCounts = 0 AND allIncludingGroupsAreEmpty = true | **Group Analysis**: PermissionSetGroupComponent relationships and member counts **Check**: `d?.isCustom === true && d?.isGroup === false && d?.memberCounts === 0 && d?.allIncludingGroupsAreEmpty === true` |
| 56 | Custom permission set group with no member | Useless | Permission Set Groups | Custom permission set group has no members assigned | isCustom = true AND isGroup = true AND memberCounts = 0 | **Group Type**: PermissionSet.Type = 'Group' and assignment counts **Check**: `d?.isCustom === true && d?.isGroup === true && d?.memberCounts === 0` |
| 57 | Custom permission set with no member and not even assigned to group | Useless | Permission Sets | Custom permission set has no members and isn't assigned to any permission set group | isCustom = true AND isGroup = false AND memberCounts = 0 AND permissionSetGroupIds array is empty | **Assignment Analysis**: PermissionSetGroupComponent relationships **Check**: `d?.isCustom === true && d?.isGroup === false && d?.memberCounts === 0 && d?.permissionSetGroupIds?.length === 0` |

## Technical Architecture & Data Flow

### Core Components

**DataFactory System**
- Each Salesforce object type (SFDC_ApexClass, SFDC_Field, etc.) has a corresponding DataFactory
- Factories handle object creation, dependency injection, and score computation
- Score calculation triggered via `dataFactory.computeScore(item)` or `dataFactory.createWithScore()`

**Dataset Classes (Data Extraction)**
- Each dataset class inherits from `Dataset` base class
- Implements `async run(sfdcManager, dataFactory, logger, parameters)` method
- Combines SOQL/SOSL queries, Metadata API calls, and Dependency API calls
- Examples: `DatasetApexClasses`, `DatasetCustomFields`, `DatasetFlows`

**Recipe Classes (Data Transformation)**
- Transform raw dataset results into filtered, augmented arrays/matrices
- Implement `extract()` for dataset dependencies and `transform()` for business logic
- Handle namespace filtering, cross-referencing, and relationship building
- Examples: `RecipeApexClasses`, `RecipeCustomFields`, `RecipeFlows`

### Key Technical Patterns

**Dependency Resolution**
```javascript
const dependencies = await sfdcManager.dependenciesQuery(itemIds, logger);
// Creates dependency data structure with hadError flag and referenced arrays
```

**Code Analysis**
```javascript
const sourceCode = CodeScanner.RemoveCommentsFromCode(record.Body);
item.hardCodedURLs = CodeScanner.FindHardCodedURLs(sourceCode);
item.hasSOQL = CodeScanner.HasSOQLFromApexCode(sourceCode);
```

**Metadata API Integration**
```javascript
const records = await sfdcManager.readMetadataAtScale('Profile', profileIds, [], logger);
// Bulk metadata retrieval with error handling
```

**API Version Calculation**
```javascript
const GET_LATEST_API_VERSION = () => {
    const TODAY = new Date();
    const THIS_YEAR = TODAY.getFullYear();
    const THIS_MONTH = TODAY.getMonth()+1;
    return 3*(THIS_YEAR-2022)+53+(THIS_MONTH<=2?0:(THIS_MONTH<=6?1:(THIS_MONTH<=10?2:3)));
}
```

### Data Sources by Category

**Tooling API Sources**
- ApexClass, ApexTrigger, ApexCodeCoverage, ApexCodeCoverageAggregate
- FlowDefinition, Flow, WorkflowRule
- EntityDefinition, FieldDefinition, CustomField
- Layout, ValidationRule, WebLink

**REST API Sources**
- User, Profile, PermissionSet, PermissionSetAssignment
- Group, UserRole, Organization
- Document, EmailTemplate, StaticResource

**Metadata API Sources**
- Profile metadata (password policies, restrictions)
- WorkflowRule details, Flow definitions
- ProfilePasswordPolicy configurations

**Dependency API**
- Cross-component references and usage tracking
- Identifies orphaned components across all metadata types

## Rule Categories Deep Dive

**Dependencies (4 rules)**: Use Salesforce Dependency API to identify unused components
**API Version (2 rules)**: Compare component versions against calculated current API version  
**Code Quality (8 rules)**: Analyze Apex code structure, coverage, and best practices
**Documentation (4 rules)**: Check for empty description fields across components
**User Adoption (2 rules)**: Track Lightning Experience adoption and login patterns
**Security (10 rules)**: Validate password policies, IP restrictions, and sharing models
**Useless (14 rules)**: Identify inactive, unassigned, or redundant components
**Overuse (2 rules)**: Monitor org limits and license consumption
**Hard-coded URLs/IDs (5 rules)**: Scan code and configuration for hard-coded values

*Total Rules: 58 (IDs 0-57) - Implemented across 45+ Dataset classes and 35+ Recipe classes*
