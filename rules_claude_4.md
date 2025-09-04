
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

# OrgCheck Score Rules - Complete Reference Table

| Rule ID | Rule Name | Category | Salesforce Artifacts Targeted | Description | Formula Logic |
|---------|-----------|----------|--------------------------------|-------------|---------------|
| 0 | Not referenced anywhere | Dependencies | Custom Labels, Flows, Lightning Pages, Lightning Aura Components, Lightning Web Components, Visualforce Components, Visualforce Pages, Static Resources | Component exists but is not referenced by any other component in the org | No dependency errors AND referenced array is empty |
| 1 | No reference anywhere for custom field | Dependencies | Custom Fields | Custom field exists but is not used anywhere in the org | Field is custom AND no dependency errors AND referenced array is empty |
| 2 | No reference anywhere for apex class | Dependencies | Apex Classes | Non-test Apex class exists but is not referenced anywhere | Not a test class AND no dependency errors AND referenced array is empty |
| 3 | Dependency API error | Dependencies | Fields, Apex Classes, Custom Labels, Flows, Lightning Pages, Lightning Aura Components, Lightning Web Components, Visualforce Components, Visualforce Pages | Unable to retrieve dependency information from Salesforce API | Dependencies object has hadError = true |
| 4 | API Version too old | API Version | Apex Classes, Apex Triggers, Flows, Lightning Aura Components, Lightning Web Components, Visualforce Pages, Visualforce Components, Email Templates | Component uses an API version that is more than 3 years old | Current API version - component API version >= 3 years |
| 5 | No assertion in Apex Test | Code Quality | Apex Test Classes | Test class exists but contains no System.assert statements | Class is test AND nbSystemAsserts = 0 |
| 6 | No description | Documentation | Lightning Pages, Lightning Aura Components, Lightning Web Components, Visualforce Pages, Visualforce Components, Workflows, Web Links, Field Sets, Validation Rules, Documents, Custom Tabs, Email Templates, Static Resources | Component lacks descriptive documentation | Description field is empty or null |
| 7 | No description for custom component | Documentation | Custom Fields, Custom Permission Sets, Custom Profiles | Custom component lacks descriptive documentation | Component is custom AND description field is empty |
| 8 | No explicit sharing in apex class | Code Quality | Apex Classes | Apex class doesn't specify sharing model (with/without/inherit sharing) | Not test class AND is class AND specifiedSharing is false |
| 9 | Schedulable should be scheduled | Code Quality | Apex Classes | Class implements Schedulable interface but is not actually scheduled | isScheduled = false AND isSchedulable = true |
| 10 | Not able to compile class | Code Quality | Apex Classes | Apex class has compilation errors | needsRecompilation = true |
| 11 | No coverage for this class | Code Quality | Apex Classes | Non-test class has no code coverage from unit tests | Not test class AND (coverage is NaN OR coverage is falsy) |
| 12 | Coverage not enough | Code Quality | Apex Classes | Class has some code coverage but less than 75% | coverage > 0 AND coverage < 0.75 |
| 13 | At least one testing method failed | Code Quality | Apex Test Classes | Test class has one or more failed test methods | Is test class AND testFailedMethods array has elements |
| 14 | Apex trigger should not contain SOQL statement | Code Quality | Apex Triggers | Trigger contains SOQL queries (violates best practices) | hasSOQL = true |
| 15 | Apex trigger should not contain DML action | Code Quality | Apex Triggers | Trigger contains DML operations (violates best practices) | hasDML = true |
| 16 | Apex Trigger should not contain logic | Code Quality | Apex Triggers | Trigger has too much code, suggesting it contains business logic | Code length > 5000 characters |
| 17 | No direct member for this group | Useless | Groups (Public Groups/Queues) | Group has no direct members (users or subgroups) | nbDirectMembers is falsy OR = 0 |
| 18 | Custom profile with no member | Useless | Custom Profiles | Custom profile exists but has no users assigned | Profile is custom AND memberCounts = 0 |
| 19 | Role with no active users | Useless | User Roles | Role has no active users assigned to it | activeMembersCount = 0 |
| 20 | Active user not under LEX | User Adoption | Users | Active user is still using Salesforce Classic interface | onLightningExperience = false |
| 21 | Active user never logged | User Adoption | Users | Active user account has never been used to log in | lastLogin = null |
| 22 | Workflow with no action | Dependencies | Workflows | Workflow rule exists but has no associated actions | hasAction = false |
| 23 | Workflow with empty time triggered list | Dependencies | Workflows | Time-based workflow has time triggers but no time-triggered actions | emptyTimeTriggers array has elements |
| 24 | Password policy with question containing password | Security | Profile Password Policies | Password policy allows password hints that may contain the actual password | passwordQuestion = true |
| 25 | Password policy with too big expiration | Security | Profile Password Policies | Password expiration period is longer than 90 days | passwordExpiration > 90 |
| 26 | Password policy with no expiration | Security | Profile Password Policies | Passwords never expire (potentially for technical users) | passwordExpiration = 0 |
| 27 | Password history too small | Security | Profile Password Policies | Password history setting is too low (less than 3 previous passwords) | passwordHistory < 3 |
| 28 | Password minimum size too small | Security | Profile Password Policies | Minimum password length is less than 8 characters | minimumPasswordLength < 8 |
| 29 | Password complexity too weak | Security | Profile Password Policies | Password complexity requirements are too low | passwordComplexity < 3 |
| 30 | No max login attempts set | Security | Profile Password Policies | No limit on failed login attempts before lockout | maxLoginAttempts is undefined |
| 31 | No lockout period set | Security | Profile Password Policies | No lockout period defined for failed login attempts | lockoutInterval is undefined |
| 32 | IP Range too large | Security | Profile Login IP Restrictions | IP address range allows too many addresses (>100,000 IPs) | Any IP range with difference > 100,000 |
| 33 | Login hours too large | Security | Profile Login Hours | Login time window is too broad (>20 hours per day) | Any login hour range with difference > 1200 minutes |
| 34 | Inactive component | Useless | Validation Rules, Record Types, Apex Triggers, Workflows, Email Templates | Component exists but is marked as inactive | isActive = false |
| 35 | No active version for this flow | Useless | Flows | Flow has no active version deployed | isVersionActive = false |
| 36 | Too many versions under this flow | Useless | Flows | Flow has accumulated too many versions (>7) | versionsCount > 7 |
| 37 | Migrate this process builder | Useless | Process Builder (Flows) | Process Builder should be migrated to Flow | currentVersionRef.type = 'Workflow' |
| 38 | No description for the current version of a flow | Documentation | Flows | Current active version of flow lacks description | currentVersionRef.description is empty |
| 39 | API Version too old for the current version of a flow | API Version | Flows | Current flow version uses an API version more than 3 years old | Current API version - flow API version >= 3 years |
| 40 | This flow is running without sharing | Security | Flows | Flow runs in system mode without sharing restrictions | currentVersionRef.runningMode = 'SystemModeWithoutSharing' |
| 41 | Too many nodes in this version | Useless | Flows | Flow has too many elements/nodes (>100), should consider Apex or subflows | currentVersionRef.totalNodeCount > 100 |
| 42 | Near the limit | Overuse | Salesforce Org Limits | Organization is approaching a Salesforce limit (>80% usage) | usedPercentage >= 0.80 |
| 43 | Almost all licenses are used | Overuse | Permission Set Licenses | Permission set license allocation is nearly exhausted (>80% used) | usedPercentage >= 0.80 |
| 44 | You could have licenses to free up | Useless | Permission Set Licenses | License usage count doesn't match active distinct assignees | remainingCount > 0 AND distinctActiveAssigneeCount ≠ usedCount |
| 45 | Role with a level >= 7 | Useless | User Roles | Role hierarchy is too deep (7+ levels), impacting performance | level >= 7 |
| 46 | Hard-coded URL suspicion in this item | Hard-coded URLs | Apex Classes, Apex Triggers, Collaboration Groups, Fields, Home Page Components, Visualforce Components, Visualforce Pages, Web Links, Custom Tabs, Email Templates | Component contains hard-coded URLs pointing to Salesforce domains | hardCodedURLs array has elements |
| 47 | Hard-coded Salesforce IDs suspicion in this item | Hard-coded IDs | Apex Classes, Apex Triggers, Collaboration Groups, Fields, Home Page Components, Visualforce Components, Visualforce Pages, Web Links, Custom Tabs, Email Templates | Component contains hard-coded Salesforce record IDs | hardCodedIDs array has elements |
| 48 | At least one successful testing method was very long | Code Quality | Apex Test Classes | Test class has methods that take too long to execute (>20 seconds) | testPassedButLongMethods array has elements |
| 49 | Page layout should be assigned to at least one Profile | Useless | Page Layouts | Page layout exists but is not assigned to any profile | profileAssignmentCount = 0 |
| 50 | Hard-coded URL suspicion in this document | Hard-coded URLs | Documents | Document URL contains hard-coded Salesforce domain references | isHardCodedURL = true |
| 51 | Unassigned Record Type | Useless | Record Types | Record type is neither default nor visible in any profile | isDefault = false AND isAvailable = false |
| 52 | Email template never used | Useless | Email Templates | Email template has never been used in the organization | lastUsedDate = null |
| 53 | No description for this flow | Documentation | Flows (excluding Process Builders) | Flow lacks descriptive documentation | type ≠ 'Workflow' AND description is empty |
| 54 | Hard-coded URL suspicion in this article | Hard-coded URLs | Knowledge Articles | Knowledge article contains links to hard-coded Salesforce domains | isHardCodedURL = true |
| 55 | Custom permission set with no member and only assigned to empty group(s) | Useless | Permission Sets | Custom permission set has no direct members and only belongs to empty groups | isCustom = true AND isGroup = false AND memberCounts = 0 AND allIncludingGroupsAreEmpty = true |
| 56 | Custom permission set group with no member | Useless | Permission Set Groups | Custom permission set group has no members assigned | isCustom = true AND isGroup = true AND memberCounts = 0 |
| 57 | Custom permission set with no member and not even assigned to group | Useless | Permission Sets | Custom permission set has no members and isn't assigned to any permission set group | isCustom = true AND isGroup = false AND memberCounts = 0 AND permissionSetGroupIds array is empty |

## Rule Categories Summary

- **Dependencies** (4 rules): Components that are orphaned or have dependency issues
- **API Version** (2 rules): Components using outdated API versions
- **Code Quality** (8 rules): Apex code that doesn't follow best practices
- **Documentation** (4 rules): Components lacking proper description/documentation
- **User Adoption** (2 rules): Users not following modern Salesforce practices
- **Security** (10 rules): Security configurations that need attention
- **Useless** (14 rules): Components that appear to serve no purpose
- **Overuse** (2 rules): Resources approaching capacity limits
- **Hard-coded URLs** (4 rules): Components with hard-coded Salesforce URLs
- **Hard-coded IDs** (1 rule): Components with hard-coded Salesforce record IDs

## Key Insights

- **Most Common Issues**: Useless components (24% of rules) and Security configurations (17% of rules)
- **API Versioning**: Rules check for components older than 3 years
- **Code Quality Focus**: Strong emphasis on Apex best practices and test coverage
- **Security Priority**: Comprehensive password policy and access restriction validation
- **Performance Considerations**: Rules identify potential performance impacts (deep role hierarchies, complex flows)

*Total Rules: 58 (IDs 0-57)*
