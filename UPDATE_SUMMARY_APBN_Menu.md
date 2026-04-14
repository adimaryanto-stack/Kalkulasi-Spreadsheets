# Update Summary - Sub Menu APBN pertahun Added

**Date**: April 13, 2026  
**Version**: 2.1 (Updated from 2.0)

---

## 🎯 Changes Made

### Menu Structure Updated

**Previous (v2.0)**:
```
📊 Dashboard (Main)
📍 Sub Menu Provinsi
🏛️ Sub Menu Kabupaten/Kota
🎓 Sub Menu Jenjang Pendidikan (5 sub-menus)
👥 Menu User Manager
```

**Current (v2.1)** ← NEW!:
```
📊 Dashboard (Main)
💰 Sub Menu APBN pertahun          ← ADDED!
📍 Sub Menu Provinsi
🏛️ Sub Menu Kabupaten/Kota
🎓 Sub Menu Jenjang Pendidikan
   ├─ 🎯 Universitas
   ├─ 🎯 SMA
   ├─ 🎯 SMP
   ├─ 🎯 SD
   └─ 🎯 PAUD
👥 Menu User Manager
```

---

## 📋 What is Sub Menu APBN pertahun?

**Purpose**: Master data management untuk tahun anggaran APBN Pendidikan

**URL**: `/dashboard/apbn`

**Features**:
1. ✅ **Manage Tahun Anggaran** - Create, edit, delete tahun anggaran
2. ✅ **Status Management** - DRAFT, ACTIVE, CLOSED
3. ✅ **Historical Data** - View data tahun-tahun sebelumnya (2020-2026)
4. ✅ **Activation Control** - Only 1 tahun ACTIVE at a time
5. ✅ **Spreadsheet Interface** - Editable cells untuk total anggaran

**Example Data** (Sesuai Excel Anda):
| Tahun | Total Anggaran | Status |
|-------|----------------|--------|
| 2020  | 473.7 T        | CLOSED |
| 2021  | 472.6 T        | CLOSED |
| 2022  | 472.6 T        | CLOSED |
| 2023  | 612.2 T        | CLOSED |
| 2024  | 665.0 T        | CLOSED |
| 2025  | 722.6 T        | CLOSED |
| 2026  | 769.1 T        | **ACTIVE** |
| 2027  | 0              | DRAFT |

---

## 🔄 Integration dengan Menu Lain

### 1. Dashboard
- Dropdown "Tahun Anggaran" → Mengambil list dari APBN menu
- Default tahun = ACTIVE tahun (2026)

### 2. Provinsi
- Filter by tahun → Menggunakan ACTIVE tahun atau pilih dari dropdown
- Total alokasi provinsi tidak boleh > Total APBN tahun tersebut

### 3. Kabupaten/Kota & Jenjang
- Inherit tahun dari Provinsi yang dipilih
- Semua data terikat ke tahun anggaran tertentu

---

## 📄 Documents Updated

### 1. ✅ PRD v2 - Section 2.2 Added
**File**: `PRD_v2_Transparansi_Anggaran_Spreadsheet.md`

**New Section**:
- **2.2 Sub Menu: APBN pertahun** (full specification)
- Spreadsheet layout
- Features (editable cells, status management, actions)
- API endpoints
- Use cases (Create, Activate, Close, View)
- Integration dengan menu lain
- Example data

**Updated Sections**:
- Section numbering: 2.3 Provinsi, 2.4 Kabkota, 2.5 Jenjang, 2.6 Users

---

### 2. ✅ MVP Roadmap v2 - Sprint 1 Updated
**File**: `MVP_Roadmap_v2_Spreadsheet.md`

**Added in Sprint 1**:
- **Day 11-12: APBN Pertahun Page**
  - Create APBN page component
  - Implement DataGrid with editable cells
  - Create API endpoints (GET, POST, PUT, DELETE)
  - Implement status management (DRAFT/ACTIVE/CLOSED)
  - Add validation: only 1 ACTIVE at a time
  - Actions: Add, Activate, Close, Delete, View

**Sprint 1 Deliverables Updated**:
- ✅ APBN pertahun page functional ← NEW!

**Sidebar Component Updated**:
- Added APBN menu item dengan DollarSign icon

---

### 3. ✅ System Diagrams v2 - Updated
**File**: `System_Diagrams_v2_Spreadsheet.md`

**Diagram 2: Menu Structure**
- Added APBN node dengan styling (green color: #059669)
- Added APBN sub-nodes: APBNTable, APBNActions

**Diagram 4: User Interaction Flow**
- Added APBN flow:
  - MenuSelect → PageAPBN
  - APBNActions → AddTahun, ActivateTahun, CloseTahun, ViewTahun
  - Flow logic for activate & close

---

## 🎨 Visual Design

### Spreadsheet Interface
```
┌────────────────────────────────────────────────────────────┐
│ APBN PENDIDIKAN PERTAHUN          [+ Add Tahun] [⬇️ Export] │
├───┬──────┬────────────────────┬──────────┬─────────────────┤
│ No│ Tahun│ Total Anggaran     │ Status   │ Actions         │
├───┼──────┼────────────────────┼──────────┼─────────────────┤
│ 1 │ 2020 │ 473,700,000,000,000│ CLOSED   │ [View]          │
│ 2 │ 2021 │ 472,600,000,000,000│ CLOSED   │ [View]          │
│ ...                                                         │
│ 7 │ 2026 │ 769,100,000,000,000│ ACTIVE ✓ │ [Edit] [Close]  │
│ 8 │ 2027 │ 0                  │ DRAFT    │ [Edit] [Delete] │
└───┴──────┴────────────────────┴──────────┴─────────────────┘
```

### Status Color Coding
- 🟢 **ACTIVE** - Green text, bold (tahun yang sedang berjalan)
- 🔵 **DRAFT** - Blue text (tahun baru, belum diaktifkan)
- ⚫ **CLOSED** - Gray text (tahun yang sudah selesai, archived)

---

## 🔑 Key Features

### 1. Status Management
**DRAFT → ACTIVE → CLOSED**

- **DRAFT**: New tahun, can edit & delete
- **ACTIVE**: Current tahun, can edit but cannot delete (only 1 ACTIVE allowed)
- **CLOSED**: Archived tahun, read-only (untuk audit trail)

### 2. Actions by Status

| Status | Edit Total | Delete | Activate | Close | View |
|--------|-----------|---------|----------|-------|------|
| DRAFT  | ✅        | ✅      | ✅       | ❌    | ✅   |
| ACTIVE | ✅        | ❌      | ❌       | ✅    | ✅   |
| CLOSED | ❌        | ❌      | ❌       | ❌    | ✅   |

### 3. Validation Rules
- ✅ Tahun must be unique (no duplicate)
- ✅ Total Anggaran > 0
- ✅ Only 1 tahun can have status ACTIVE
- ✅ Cannot delete tahun yang sudah ada alokasi provinsi
- ✅ SUM(alokasi_provinsi) <= total_anggaran

---

## 🛠️ Technical Implementation

### Database Table: `tahun_anggaran`
```sql
CREATE TABLE tahun_anggaran (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tahun INT UNIQUE NOT NULL,
  total_anggaran DECIMAL(15,2) NOT NULL,
  status VARCHAR(20) DEFAULT 'DRAFT',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### API Endpoints
```
GET    /api/tahun-anggaran           - List all tahun
GET    /api/tahun-anggaran/:id       - Get detail
POST   /api/tahun-anggaran           - Create (status: DRAFT)
PUT    /api/tahun-anggaran/:id       - Update total or status
PUT    /api/tahun-anggaran/:id/activate - Set ACTIVE (close others)
PUT    /api/tahun-anggaran/:id/close    - Set CLOSED
DELETE /api/tahun-anggaran/:id       - Delete (only DRAFT)
```

### React Component
```typescript
// app/dashboard/apbn/page.tsx
<DataGrid
  columns={[
    { key: 'tahun', name: 'Tahun' },
    { key: 'total_anggaran', name: 'Total Anggaran', editable: true },
    { key: 'status', name: 'Status' },
    { key: 'actions', name: 'Actions', renderCell: ActionsCell }
  ]}
  rows={data}
  onRowsChange={handleUpdate}
/>
```

---

## 📊 Data Flow

### Flow 1: Activate Tahun Baru
```
User clicks [Activate] pada tahun 2027
  ↓
System check: Ada tahun ACTIVE lain? (2026)
  ↓
Confirm: "Close tahun 2026 dan activate 2027?"
  ↓ User confirms
Database: UPDATE tahun_anggaran SET status='CLOSED' WHERE status='ACTIVE'
  ↓
Database: UPDATE tahun_anggaran SET status='ACTIVE' WHERE id=2027
  ↓
Frontend refresh: 2027 now ACTIVE, 2026 now CLOSED
  ↓
Dashboard & Provinsi now use 2027 data
```

### Flow 2: View Historical Data
```
User clicks [View] pada tahun 2025 (CLOSED)
  ↓
Navigate to: /dashboard/provinsi?tahun=2025
  ↓
Provinsi page loads with filter tahun=2025
  ↓
User can see historical data (read-only)
  ↓
Can export to Excel for audit
```

---

## 🎯 Benefits

### 1. **Multi-Year Support**
- Dapat manage multiple tahun anggaran
- Historical data preserved untuk audit

### 2. **Clean Year Transition**
- Smooth transition dari tahun lama ke tahun baru
- Old data automatically archived (CLOSED)

### 3. **Data Integrity**
- Only 1 ACTIVE tahun at a time
- Validation: Total provinsi <= Total APBN

### 4. **Audit Trail**
- CLOSED tahun cannot be modified
- Historical data preserved forever
- Can view & export anytime

---

## 📝 Development Timeline

### Sprint 1, Day 11-12 (Week 2)
- [x] Create APBN page component
- [x] Implement DataGrid
- [x] Create API endpoints
- [x] Implement status management
- [x] Add validation
- [x] Test activate, close, delete

**Estimated Time**: 2 days (16 hours)

---

## ✅ Checklist for Implementation

### Backend
- [ ] Create `tahun_anggaran` table
- [ ] Seed data tahun 2020-2026
- [ ] Create API routes (GET, POST, PUT, DELETE)
- [ ] Implement status management logic
- [ ] Add validation: only 1 ACTIVE
- [ ] Add validation: cannot delete if has alokasi

### Frontend
- [ ] Create APBN page component
- [ ] Implement DataGrid with editable cells
- [ ] Create actions (Add, Activate, Close, Delete, View)
- [ ] Add status color coding
- [ ] Test inline editing
- [ ] Test navigation to Provinsi

### Integration
- [ ] Dashboard dropdown uses APBN data
- [ ] Provinsi filter uses APBN data
- [ ] Validation: SUM provinsi <= total APBN
- [ ] Test year transition (activate → close)

---

## 🚀 Next Steps

1. ✅ **Review Updated Docs** - Read PRD, Roadmap, Diagrams
2. ✅ **Implement Sprint 1, Day 11-12** - Build APBN page
3. ✅ **Test End-to-End** - Create → Activate → Close → View
4. ✅ **Continue to Sprint 2** - Provinsi & Kabkota with year filter

---

**Status**: ✅ DOCUMENTATION UPDATED  
**Ready for Development**: YES  
**Version**: 2.1  
**Last Updated**: April 13, 2026
