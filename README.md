eastcharmer.local (Domain Root)
│
├── FILESERVER01 (Windows Server 2025 DC / DNS / File Server)
│   └── Shared Folder: C:\CompanyData (\\FILESERVER01\CompanyData)
│       ├── HR
│       ├── IT
│       ├── Finance
│       └── Public
│
└── USA (Top-Level OU)
    ├── Users (Sub-OU)
    │   ├── IT (Sub-OU)
    │   ├── HR (Sub-OU)
    │   └── Finance (Sub-OU)
    └── Computers (Sub-OU)
        └── CLIENT01 (Windows 11 Pro Endpoint)
