flowchart TD
    Start[Start] --> SignIn[Sign In Page]
    SignIn --> AuthAPI[Auth API Endpoint]
    AuthAPI --> Session[Create Session]
    Session --> RoleDecision[Determine User Role]
    RoleDecision -->|kurikulum| KurikulumDash[Kurikulum Dashboard]
    RoleDecision -->|guru| GuruDash[Guru Dashboard]
    RoleDecision -->|siswa| SiswaDash[Siswa Dashboard]
    subgraph Kurikulum Portal
        KurikulumDash --> KManageCurriculum[Manage Curriculum]
        KurikulumDash --> KManageUsers[Manage Users]
    end
    subgraph Guru Portal
        GuruDash --> GGradeMgmt[Grade Management]
        GuruDash --> GViewSchedule[View Schedule]
        GuruDash --> GUploadMaterials[Upload Materials]
    end
    subgraph Siswa Portal
        SiswaDash --> SViewGrades[View Grades]
        SiswaDash --> SViewSchedule[View Schedule]
        SiswaDash --> SProfile[View Profile]
    end