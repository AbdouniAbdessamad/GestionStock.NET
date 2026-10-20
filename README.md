# GestionStock.NET

> A Windows desktop application for managing a store's products, suppliers and customers, built with C# WinForms, Entity Framework 6 and SQL Server.

![C#](https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=csharp&logoColor=white)
![.NET Framework](https://img.shields.io/badge/.NET_Framework-4.7.2-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![Windows Forms](https://img.shields.io/badge/Windows_Forms-0078D6?style=for-the-badge&logo=windows&logoColor=white)
![Entity Framework](https://img.shields.io/badge/Entity_Framework-6.2-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL_Server-Express-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)

## Overview

GestionStock.NET is a stock management desktop application for a small retail business. It replaces manual record keeping with a single interface where staff can register products (reference, quantity, shelf location, description), keep a directory of suppliers and customers, and print a product sheet. The application follows a simple layered design (presentation, business logic, reporting) on top of a SQL Server database accessed through Entity Framework 6 (database-first). The user interface is in French.

## Key Features

- **Product management**: add, edit and delete products (name, reference, quantity, shelf number, description), with duplicate detection on name + reference and multi-row deletion with confirmation.
- **Product search**: live filtering of the product list by name.
- **Product report**: generates a printable/exportable product sheet with the RDLC Report Viewer.
- **Supplier management**: add, edit and delete suppliers (name, address, phone), with duplicate-name check.
- **Customer management**: add, edit and delete customers (name, address, phone), with duplicate-name check.
- **Input validation**: required-field checks, numeric-only input for phone and quantity fields, and placeholder text in form fields.
- **User login form**: checks a username and password against the `utilisateur` table (opened from the settings panel of the main menu).
- **Navigation shell**: main window with a collapsible side menu that hosts each module as a docked user control.

## Tech Stack

| Layer | Technology |
| --- | --- |
| Language | C# |
| Framework | .NET Framework 4.7.2, Windows Forms |
| Data access | Entity Framework 6.2.0 (database-first, EDMX model) |
| Database | Microsoft SQL Server (Express) |
| Reporting | Microsoft ReportViewer WinForms 15 (RDLC, package `150.1404.0`) |
| IDE | Visual Studio 2017 (solution format v15) |

Other NuGet packages referenced in `packages.config`: ExcelDataReader 3.6.0 (used by an experimental Excel preview form), LiveCharts 0.9.7 (referenced, not used in the current code) and Microsoft.SqlServer.Types 14.0.

## Architecture / Project Structure

```
GestionStock.NET/
├── appGestion.sln
├── GESTION.bak                 # SQL Server backup of the GESTION database
└── appGestion/
    ├── Program.cs              # Entry point (opens the main menu)
    ├── App.config              # Connection strings + EF configuration
    ├── DbStock.edmx            # Entity Framework model (database-first)
    ├── DbStock.Context.cs      # DbStockContext (DbSets for each table)
    ├── Produit.cs, Client.cs, Fournisseur.cs, utilisateur.cs   # Generated entities
    ├── BL/                     # Business logic: CRUD classes per entity + login check
    ├── PL/                     # Presentation: forms (add/edit dialogs, login, menu) and list user controls
    ├── RAP/                    # Reporting: report viewer form + RPT_Produit.rdlc
    └── Resources/              # Icons and images used by the UI
```

- **PL (Presentation Layer)**: `FRM_Menu` is the main window. Each module (products, suppliers, customers) is a singleton `UserControl` with a data grid, docked into the menu's content panel. Add and edit share one dialog per entity.
- **BL (Business Layer)**: `CLS_Produit`, `CLS_Fournisseur` and `CLS_Client` wrap create/update/delete operations through `DbStockContext`; `CLS_Connexion` validates logins.
- **RAP (Reporting)**: `FRM_Rapport` hosts a `ReportViewer` that renders `RPT_Produit.rdlc` with report parameters for the selected product.

## Database

Database name: `GESTION` (SQL Server). Schema, as defined in the EDMX storage model:

| Table | Columns |
| --- | --- |
| `Produit` | `ID_Produit` (int, identity, PK), `Nom_Produit`, `Reference_Produit`, `Quantite_Produit`, `Rayon_Produit`, `Designation_Produit` (all `nvarchar(250)`) |
| `Fournisseur` | `ID_Fournisseur` (int, identity, PK), `Nom_Fournisseur`, `Adresse_Fournisseur`, `Téléphone_Fournisseur` (`nvarchar(250)`) |
| `Client` | `ID_Client` (int, identity, PK), `Nom_Client`, `Adresse_Client`, `Téléphone_Client` (`nvarchar(250)`) |
| `utilisateur` | `NomUtilisateur` (`nvarchar(250)`, PK), `MotdePasse` (`nvarchar(250)`) |

The tables are independent (no foreign keys between them).

## Getting Started

### Prerequisites

- Windows
- Visual Studio 2017 or later with the **.NET desktop development** workload
- .NET Framework 4.7.2 Developer Pack
- SQL Server or SQL Server Express, plus SQL Server Management Studio (SSMS)
- Microsoft RDLC Report Designer extension (only needed to edit the report)

### 1. Clone

```bash
git clone https://github.com/AbdouniAbdessamad/GestionStock.NET.git
cd GestionStock.NET
```

### 2. Create the database

**Option A: restore the backup.** In SSMS, right-click **Databases → Restore Database…**, choose **Device**, select `GESTION.bak` from the repository root and restore it as `GESTION`.

**Option B: create the schema manually.**

```sql
CREATE DATABASE GESTION;
GO
USE GESTION;
GO
CREATE TABLE Produit (
    ID_Produit INT IDENTITY(1,1) PRIMARY KEY,
    Nom_Produit NVARCHAR(250) NOT NULL,
    Reference_Produit NVARCHAR(250) NOT NULL,
    Quantite_Produit NVARCHAR(250) NOT NULL,
    Rayon_Produit NVARCHAR(250) NOT NULL,
    Designation_Produit NVARCHAR(250) NOT NULL
);
CREATE TABLE Fournisseur (
    ID_Fournisseur INT IDENTITY(1,1) PRIMARY KEY,
    Nom_Fournisseur NVARCHAR(250) NOT NULL,
    Adresse_Fournisseur NVARCHAR(250) NOT NULL,
    Téléphone_Fournisseur NVARCHAR(250) NOT NULL
);
CREATE TABLE Client (
    ID_Client INT IDENTITY(1,1) PRIMARY KEY,
    Nom_Client NVARCHAR(250) NOT NULL,
    Adresse_Client NVARCHAR(250) NOT NULL,
    Téléphone_Client NVARCHAR(250) NOT NULL
);
CREATE TABLE utilisateur (
    NomUtilisateur NVARCHAR(250) PRIMARY KEY,
    MotdePasse NVARCHAR(250) NOT NULL
);
```

Add a user to the `utilisateur` table if you want to test the login form.

### 3. Configure the connection string

Edit `appGestion/App.config` and replace the server name in **both** connection strings with your own SQL Server instance:

```xml
<add name="DbStockContext"
     connectionString="metadata=res://*/DbStock.csdl|res://*/DbStock.ssdl|res://*/DbStock.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=YOUR_SERVER\SQLEXPRESS;initial catalog=GESTION;integrated security=True;MultipleActiveResultSets=True;App=EntityFramework&quot;"
     providerName="System.Data.EntityClient"/>
<add name="appGestion.Properties.Settings.GESTIONConnectionString"
     connectionString="Data Source=YOUR_SERVER\SQLEXPRESS;Initial Catalog=GESTION;Integrated Security=True"
     providerName="System.Data.SqlClient"/>
```

### 4. Build and run

1. Open `appGestion.sln` in Visual Studio.
2. Restore NuGet packages (**right-click the solution → Restore NuGet Packages**).
3. Press **F5** to build and run. The application opens on the main menu.

## Possible Improvements

- Hash passwords instead of storing and comparing them in plain text, and require login before the main menu is shown.
- Store quantities as numeric columns and add stock movements (entries and exits) linked to suppliers and customers.
- Extend search to all fields (the "ID" option is not yet wired) and to the supplier and customer lists.
- Clean the repository: add a Visual Studio `.gitignore`, remove committed `bin/`, `obj/`, `.vs/` and `packages/` folders, and drop unused dataset files and NuGet packages.

## Author

**Abdessamad Abdouni**, Information Systems Engineering student at SUPMTI Rabat, Morocco
GitHub: [github.com/AbdouniAbdessamad](https://github.com/AbdouniAbdessamad)
