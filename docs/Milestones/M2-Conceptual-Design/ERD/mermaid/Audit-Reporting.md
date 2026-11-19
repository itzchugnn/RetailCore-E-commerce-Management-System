```mermaid
erDiagram
    ACCOUNT ||--o{ AUDIT_LOG : "performs (0..1 : 0..N)"
    AUDIT_LOG ||--o{ AUDIT_LOG_DETAIL : "details_changes (1..1 : 0..N)"

    REPORT_DEFINITION ||--o{ REPORT_EXECUTION : "defines_template (1..1 : 0..N)"
    ACCOUNT ||--o{ REPORT_EXECUTION : "requests_execution (1..1 : 0..N)"

    REPORT_DEFINITION ||--o{ REPORT_SNAPSHOT : "captures_period (1..1 : 0..N)"
    ACCOUNT ||--o{ REPORT_SNAPSHOT : "created_by (0..1 : 0..N)"

    REPORT_DEFINITION ||--o{ REPORT_PERMISSION : "grants_access (1..1 : 0..N)"
    ROLE ||--o{ REPORT_PERMISSION : "holds_permission (1..1 : 0..N)"

    AUDIT_LOG {
        bigint AuditLogID PK
        bigint AccountID FK
        varchar ActionType
        varchar EntityName
        varchar RecordID
        datetimeoffset ActionAt
        varchar IPAddress
        nvarchar UserAgent
        varchar Result
        nvarchar Reason
        varchar CorrelationID
        datetimeoffset RetentionUntil
    }
    AUDIT_LOG_DETAIL {
        bigint AuditLogDetailID PK
        bigint AuditLogID FK
        varchar FieldName
        nvarchar OldValue
        nvarchar NewValue
    }
    REPORT_DEFINITION {
        bigint ReportDefinitionID PK
        varchar ReportCode UK
        nvarchar ReportName
        nvarchar Description
        varchar DataSource
        varchar ReportType
        bit IsActive
        datetimeoffset CreatedAt
    }
    REPORT_EXECUTION {
        bigint ReportExecutionID PK
        bigint ReportDefinitionID FK
        bigint AccountID FK
        nvarchar Parameters
        varchar ExecutionStatus
        datetimeoffset RequestedAt
        datetimeoffset CompletedAt
        varchar ExportFormat
        nvarchar ResultLocation
        nvarchar ErrorMessage
    }
    REPORT_SNAPSHOT {
        bigint ReportSnapshotID PK
        bigint ReportDefinitionID FK
        varchar SnapshotCode UK
        datetimeoffset PeriodStart
        datetimeoffset PeriodEnd
        datetimeoffset SnapshotAt
        varchar SnapshotStatus
        nvarchar DataLocation
        bigint CreatedBy
        datetimeoffset CreatedAt
    }
    REPORT_PERMISSION {
        bigint ReportDefinitionID PK
        bigint RoleID PK
        bit CanView
        bit CanExport
        bit CanCreateSnapshot
    }
```