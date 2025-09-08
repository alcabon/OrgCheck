```code
salesforce-audit/
├── package.json                           # VSCode extension manifest with new data analysis configs
├── src/
│   ├── extension.ts                       # Main extension entry point
│   │
│   ├── providers/
│   │   ├── MetadataRetriever.ts           # FIXED: Proper org switching with connection cache
│   │   └── OrgPanelProvider.ts            # ENHANCED: Refactored with service architecture
│   │
│   ├── analyzers/
│   │   ├── SecurityAnalyzer.ts            # Security health check analysis
│   │   ├── ApexCodeCoverageAnalyzer.ts    # Async code coverage with proper ID handling
│   │   ├── ConsistentDependencyAnalyzer.ts # Memory-efficient dependency analysis
│   │   ├── EnhancedFieldsAnalyzer.ts      # Original field metadata analysis
│   │   ├── SmartFieldDataAnalyzer.ts      # 🆕 NEW: Performance-optimized data verification
│   │   ├── EnhancedFieldsAnalyzerWithData.ts # 🆕 NEW: Integration of metadata + data analysis
│   │   └── EnhancedUsageAnalyzer.ts       # MetadataComponentDependency analysis
│   │
│   ├── generators/
│   │   └── ConsistentReportGenerator.ts   # ENHANCED: Report generation with coverage data
│   │
│   ├── services/                          # 🆕 NEW: Service layer for better architecture
│   │   ├── ConfigurableAuditService.ts    # 🆕 NEW: Configuration-driven audit service
│   │   ├── DataAnalysisConfigManager.ts   # 🆕 NEW: Performance mode configuration
│   │   └── EnhancedFieldsDashboardData.ts # 🆕 NEW: Dashboard data transformation
│   │
│   ├── utils/
│   │   ├── DependencyInterfaces.ts        # FIXED: Enhanced with optional coverage support
│   │   ├── IdUtils.ts                     # ID normalization utilities
│   │   ├── MetadataDependencyLogger.ts    # Comprehensive logging for analysis
│   │   └── MetadataFileUtils.ts           # File discovery utilities
│   │
│   ├── types/
│   │   └── interfaces.ts                  # Core type definitions
│   │
│   └── media/                             # Dashboard assets
│       ├── dashboard.html                 # Dashboard UI template
│       ├── dashboard.css                  # ENHANCED: Dashboard styling with coverage badges
│       ├── dashboard.js                   # ENHANCED: Dashboard logic with data analysis
│       ├── main.html                      # Main panel template  
│       ├── main.css                       # Main panel styling
│       └── main.js                        # ENHANCED: Main panel with configuration options
│
└── out/                                   # Compiled JavaScript output
```
# Key Architecture Changes

## 🆕 NEW COMPONENTS

### 1. SmartFieldDataAnalyzer.ts
```typescript
class SmartFieldDataAnalyzer {
    // Performance-optimized field data verification
    // - Object size analysis (COUNT queries)
    // - Tiered query strategies (DIRECT/SAMPLED/AGGREGATED)
    // - Parallel processing with concurrency control
    // - Configurable thresholds and timeouts
    
    analyzeFieldDataUsage(fields: FieldInfo[]): Promise<Map<string, FieldValueAnalysis>>
    // Returns: hasData, confidence, strategy, queryTime per field
}
```

### 2. EnhancedFieldsAnalyzerWithData.ts
```typescript
class EnhancedFieldsAnalyzerWithData extends EnhancedFieldsAnalyzer {
    // Integrates metadata analysis with actual data verification
    // - Combines dependency analysis with data analysis
    // - Identifies critical scenarios (data but unused, required but empty)
    // - Provides comprehensive field recommendations
    
    analyzeFieldsWithData(objects?, includeData?, unusedOnly?): Promise<EnhancedFieldsAnalysisResult>
    integrateWithDependencyAnalysis(dependencyMap, fields): Promise<EnhancedFieldInfo[]>
}
```

### 3. Services Layer (New Architecture)
```
services/
├── ConfigurableAuditService.ts    # Main audit orchestration with configuration
├── DataAnalysisConfigManager.ts   # Performance mode management (FAST/BALANCED/THOROUGH)
└── EnhancedFieldsDashboardData.ts # Dashboard data transformation and export
```

## 🔧 ENHANCED COMPONENTS

### 1. MetadataRetriever.ts - FIXED ORG SWITCHING
```typescript
class MetadataRetriever {
    private connection: jsforce.Connection | null = null;
    private currentOrgAlias: string | null = null;        // 🆕 Track current org
    private connectionCache = new Map<string, Connection>(); // 🆕 Cache per org
    
    connectToOrg(orgAlias: string): Promise<void>         // 🔧 Proper org switching
    getConnection(): jsforce.Connection                   // 🆕 Public connection access
    forceReconnect(orgAlias: string): Promise<void>       // 🆕 Force reconnection
}
```

### 2. OrgPanelProvider.ts - SERVICE ARCHITECTURE
```typescript
// Refactored with proper service separation:
- AuditService (interface)           # Audit operations
- ReportService (interface)          # Report management  
- DashboardService (interface)       # Dashboard operations
- NotificationService (interface)    # User notifications
- Logger (interface)                 # Logging abstraction

class RefactoredOrgPanelProvider {
    // Composition over inheritance
    private auditService: AuditService;
    private reportService: ReportService;
    private dashboardService: DashboardService;
    // Clean separation of concerns
}
```

### 3. Dashboard Integration - ENHANCED UI
```
media/
├── dashboard.html    # 🔧 Enhanced with data analysis indicators
├── dashboard.css     # 🔧 New coverage badges, confidence indicators
├── dashboard.js      # 🔧 Data analysis metrics, performance charts
├── main.html         # 🔧 Configuration UI for performance modes
└── main.js           # 🔧 Configuration management integration
```

## 📊 DATA FLOW ARCHITECTURE

### Enhanced Fields Audit Flow:
```
1. ConfigurableAuditService.runFieldsAudit(orgAlias)
   ├── DataAnalysisConfigManager.configureDataAnalysis()
   │   └── Returns: FAST/BALANCED/THOROUGH configuration
   │
   ├── MetadataRetriever.connectToOrg(orgAlias)          # ✅ Fixed org switching
   │   └── Validates connection to correct org
   │
   ├── EnhancedFieldsAnalyzerWithData.analyzeFieldsWithData()
   │   ├── getEnhancedFieldMetadata()                    # Metadata analysis
   │   ├── SmartFieldDataAnalyzer.analyzeFieldDataUsage() # 🆕 Data verification
   │   │   ├── Object sizing (COUNT queries)
   │   │   ├── Strategy selection (DIRECT/SAMPLED/AGGREGATED)
   │   │   ├── Parallel field analysis with concurrency control
   │   │   └── Returns: confidence + strategy per field
   │   └── integrateWithDependencyAnalysis()             # Combine results
   │
   ├── ConsistentDependencyAnalyzer.analyzeFieldDependencies()
   │   └── MetadataComponentDependency analysis
   │
   ├── EnhancedFieldsDashboardData.transformForDashboard()
   │   └── Dashboard-ready data with risk indicators
   │
   └── ConsistentReportGenerator.generateFieldsReport()
       └── Enhanced report with data analysis insights
```

## 🎯 CRITICAL FINDINGS CLASSIFICATION

The new system identifies:

```typescript
interface CriticalFindings {
    trulyUnusedFields: Field[];        // No metadata refs + no data (✅ safe removal)
    hasDataButUnused: Field[];         // Has data but no refs (🚨 risk of data loss)  
    requiredFieldsEmpty: Field[];      // Required but no data (⚠️ data integrity)
    unnecessaryIndexes: Field[];       // Indexed but no data (🔍 performance impact)
}
```

## ⚙️ CONFIGURATION SYSTEM

New configuration options in package.json:
```json
{
  "salesforceAudit.dataAnalysis.enabled": true,
  "salesforceAudit.dataAnalysis.maxAnalysisTime": 300000,
  "salesforceAudit.dataAnalysis.smallObjectThreshold": 10000,
  "salesforceAudit.dataAnalysis.mediumObjectThreshold": 100000,
  "salesforceAudit.dataAnalysis.largeObjectThreshold": 1000000,
  "salesforceAudit.dataAnalysis.skipHugeObjects": true
}
```

## 🚀 PERFORMANCE OPTIMIZATIONS

1. **Object Size Analysis**: COUNT() queries before field analysis
2. **Tiered Strategies**: Different approaches based on object size
3. **Parallel Processing**: Controlled concurrency to avoid API limits
4. **Connection Caching**: Proper org switching without reconnection overhead
5. **Memory Efficiency**: Selective loading and processing
6. **User Control**: Performance vs accuracy tradeoffs

This architecture maintains the existing functionality while adding sophisticated data verification capabilities that scale efficiently with organization size.
