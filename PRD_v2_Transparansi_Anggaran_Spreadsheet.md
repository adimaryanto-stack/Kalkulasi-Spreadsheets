# Product Requirements Document (PRD) v2
# Sistem Transparansi Anggaran Pendidikan - Spreadsheet Interface

**Version:** 2.0 (Revised)  
**Date:** April 13, 2026  
**Project Type:** Web-Based Spreadsheet Dashboard for Education Budget Transparency  
**Tech Stack:** Next.js 14, PostgreSQL, Drizzle ORM, shadcn/ui (Table components)

---

## 1. PROJECT OVERVIEW

### 1.1 Deskripsi Aplikasi
Sistem Transparansi Anggaran Pendidikan adalah aplikasi web berbasis **spreadsheet interface** yang menampilkan dan mengelola data alokasi serta realisasi anggaran pendidikan Indonesia. Sistem ini memiliki struktur menu sederhana dengan tampilan mirip Excel/Google Sheets, di mana semua kalkulasi angka terhubung secara real-time antar menu dan database.



### 1.3 Target User
1. **Admin Kementerian**: Input dan monitoring anggaran nasional
2. **Admin Provinsi**: Monitoring anggaran provinsi
3. **Admin Kabupaten/Kota**: Monitoring anggaran kabupaten/kota
4. **Viewer**: Read-only access untuk reporting dan audit

### 1.4 Core Concept: Spreadsheet-Like Interface
- **Tampilan seperti Excel**: Table dengan rows & columns
- **Kalkulasi Real-Time**: SUM, COUNT, AVERAGE, PERCENTAGE
- **Cell Formula**: Angka terhubung antar menu (misal: Total Provinsi = SUM(Kabupaten/Kota))
- **Editable Cells**: Click to edit (inline editing)
- **Auto-Save**: Otomatis save ke database saat edit
- **Color Coding**: Conditional formatting untuk status (hijau/kuning/merah)

---

## 2. MENU STRUKTUR & FEATURES

### 2.1 Menu: Dashboard (Main Page)

**URL**: `/dashboard`

**Layout**: 
```
┌─────────────────────────────────────────────────────────────┐
│  TRANSPARANSI ANGGARAN PENDIDIKAN INDONESIA                 │
│  Tahun: [2026 ▼]                                            │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐│
│  │ Total Nominal   │  │ Total Realisasi │  │ % Penyerapan ││
│  │ 769.1 Triliun   │  │ 500.0 Triliun   │  │    65.02%    ││
│  └─────────────────┘  └─────────────────┘  └──────────────┘│
├─────────────────────────────────────────────────────────────┤
│  Ringkasan Per Jenjang Pendidikan                           │
│  ┌────────────┬───────────────┬──────────────┬────────────┐│
│  │ Jenjang    │ Nominal       │ Realisasi    │ %          ││
│  ├────────────┼───────────────┼──────────────┼────────────┤│
│  │ Universitas│ 150.0 T       │ 100.0 T      │ 66.67%     ││
│  │ SMA        │ 200.0 T       │ 130.0 T      │ 65.00%     ││
│  │ SMP        │ 180.0 T       │ 120.0 T      │ 66.67%     ││
│  │ SD         │ 200.0 T       │ 130.0 T      │ 65.00%     ││
│  │ PAUD       │  39.1 T       │  20.0 T      │ 51.15%     ││
│  └────────────┴───────────────┴──────────────┴────────────┘│
└─────────────────────────────────────────────────────────────┘
```

**Features**:
- Card summary: Total Nominal, Total Realisasi, % Penyerapan Nasional
- Table ringkasan per jenjang pendidikan
- Dropdown filter tahun anggaran
- Chart (Bar chart): Nominal vs Realisasi per jenjang
- Auto-refresh setiap data berubah di menu lain

**Database Queries**:
```sql
-- Total Nasional
SELECT 
  SUM(nominal_alokasi) as total_nominal,
  SUM(realisasi_total) as total_realisasi,
  (SUM(realisasi_total) / SUM(nominal_alokasi) * 100) as persentase
FROM alokasi_provinsi
WHERE tahun_anggaran_id = ?

-- Per Jenjang
SELECT 
  jenjang,
  SUM(nominal_alokasi) as total_nominal,
  SUM(realisasi_total) as total_realisasi,
  (SUM(realisasi_total) / SUM(nominal_alokasi) * 100) as persentase
FROM institusi_pendidikan
GROUP BY jenjang
```



**Spreadsheet Layout**:
```
┌───────────────────────────────────────────────────────────────────────────────┐
│  PROVINSI - TAHUN 2026                                        [+ Add Row] [⬇️ Export] │
├──────┬─────────────────┬─────────────────┬─────────────────┬─────────────┬──────────┤
│ No   │ Nama Provinsi   │ Nominal         │ Realisasi       │ Selisih     │ %        │
├──────┼─────────────────┼─────────────────┼─────────────────┼─────────────┼──────────┤
│  1   │ Jawa Barat      │ 20,240,000,000  │ 15,000,000,000  │ 5,240,000   │ 74.11%   │
│  2   │ Jawa Timur      │ 20,240,000,000  │ 16,500,000,000  │ 3,740,000   │ 81.52%   │
│  3   │ Jawa Tengah     │ 20,240,000,000  │ 14,000,000,000  │ 6,240,000   │ 69.15%   │
│  4   │ DKI Jakarta     │ 15,000,000,000  │ 13,500,000,000  │ 1,500,000   │ 90.00%   │
│ ...  │ ...             │ ...             │ ...             │ ...         │ ...      │
│  38  │ Papua Barat     │ 10,000,000,000  │  4,200,000,000  │ 5,800,000   │ 42.00%   │
├──────┼─────────────────┼─────────────────┼─────────────────┼─────────────┼──────────┤
│ TOTAL│ 38 Provinsi     │ 769,100,000,000 │ 500,000,000,000 │ 269,100,000 │ 65.02%   │
└──────┴─────────────────┴─────────────────┴─────────────────┴─────────────┴──────────┘
```

**Features (Spreadsheet-like)**:
1. **Editable Cells**:
   - Click cell "Nominal" atau "Realisasi" → Inline edit → Auto-save ke DB
   - Formula auto-update: `Selisih = Nominal - Realisasi`
   - Formula auto-update: `% = (Realisasi / Nominal) * 100`

2. **Row Operations**:
   - [+ Add Row]: Tambah provinsi baru (jika diperlukan)
   - Click row → Highlight → Show detail di sidebar
   - Double-click row → Navigate ke `/dashboard/kabupaten-kota?provinsi_id={id}`

3. **Conditional Formatting**:
   - % >= 80%: Background hijau (penyerapan bagus)
   - % 50-79%: Background kuning (warning)
   - % < 50%: Background merah (critical)

4. **Toolbar**:
   - 🔍 Search: Filter by nama provinsi
   - ⬇️ Export: Download as Excel (.xlsx)
   - 🔄 Refresh: Reload data dari database
   - 📊 Chart View: Toggle ke bar chart view

5. **Footer Row**:
   - TOTAL row dengan SUM formula untuk semua kolom angka

**Database Table**: `provinsi` + `alokasi_provinsi`

**API Endpoints**:
- `GET /api/provinsi?tahun={tahun}` - Fetch all provinsi data
- `PUT /api/provinsi/:id` - Update nominal/realisasi (inline edit)
- `GET /api/provinsi/:id/detail` - Detail provinsi (untuk sidebar)

**Real-Time Calculation**:
```javascript
// Frontend calculation (real-time)
const selisih = nominal - realisasi;
const persentase = (realisasi / nominal) * 100;

// Backend validation before save
if (realisasi > nominal) {
  throw new Error("Realisasi tidak boleh melebihi Nominal");
}
```



**Spreadsheet Layout dengan Filter**:
```
┌───────────────────────────────────────────────────────────────────────────────┐
│  KABUPATEN/KOTA - TAHUN 2026                                                   │
│  Filter: Provinsi [Jawa Barat ▼]                         [+ Add] [⬇️ Export]   │
├──────┬─────────────────────┬─────────────────┬─────────────────┬─────────┬────┤
│ No   │ Nama Kabupaten/Kota │ Nominal         │ Realisasi       │ Selisih │ %  │
├──────┼─────────────────────┼─────────────────┼─────────────────┼─────────┼────┤
│  1   │ Bandung             │ 500,000,000,000 │ 400,000,000,000 │ 100,000 │ 80%│
│  2   │ Bogor               │ 450,000,000,000 │ 350,000,000,000 │ 100,000 │ 78%│
│  3   │ Bekasi              │ 480,000,000,000 │ 380,000,000,000 │ 100,000 │ 79%│
│ ...  │ ...                 │ ...             │ ...             │ ...     │ ...│
│  27  │ Tasikmalaya         │ 300,000,000,000 │ 200,000,000,000 │ 100,000 │ 67%│
├──────┼─────────────────────┼─────────────────┼─────────────────┼─────────┼────┤
│ TOTAL│ 27 Kabupaten/Kota   │ 20,240,000,000  │ 15,000,000,000  │ 5,240   │ 74%│
└──────┴─────────────────────┴─────────────────┴─────────────────┴─────────┴────┘
```

**Features**:
1. **Filter by Provinsi**: Dropdown untuk pilih provinsi
2. **Editable Cells**: Sama seperti menu Provinsi
3. **Row Click**: Double-click → Navigate ke jenjang pendidikan with filter kabupaten/kota
4. **Validation**: SUM(Kabupaten/Kota) tidak boleh > Alokasi Provinsi
5. **Auto-Update Parent**: Jika data kabupaten/kota berubah → auto-update total Provinsi

**Database Linking**:
```javascript
// Saat update realisasi kabupaten/kota
const updateKabupatenKota = async (id, realisasi) => {
  // 1. Update kabupaten_kota table
  await db.update(kabupatenKota)
    .set({ realisasi_total: realisasi })
    .where(eq(kabupatenKota.id, id));
  
  // 2. Recalculate & update parent (provinsi)
  const provinsiId = await getProvinsiIdByKabkota(id);
  const totalRealisasiKabkota = await sumRealisasiByProvinsi(provinsiId);
  
  await db.update(alokasiProvinsi)
    .set({ realisasi_total: totalRealisasiKabkota })
    .where(eq(alokasiProvinsi.provinsi_id, provinsiId));
};
```



**URL**: `/dashboard/jenjang/universitas`

**Spreadsheet Layout**:
```
┌───────────────────────────────────────────────────────────────────────────────────┐
│  JENJANG: UNIVERSITAS - TAHUN 2026                                                │
│  Filter: Provinsi [Semua ▼]  Kabupaten/Kota [Semua ▼]          [+ Add] [⬇️ Export]│
├────┬──────────────────────┬─────────────────┬─────────────────┬─────────┬────┬────┤
│ No │ Nama Universitas     │ Kab/Kota        │ Nominal         │ Realisasi│ %  │NPSN│
├────┼──────────────────────┼─────────────────┼─────────────────┼─────────┼────┼────┤
│  1 │ Universitas Indonesia│ Depok           │ 2,000,000,000   │1,800,000│ 90%│1234│
│  2 │ ITB                  │ Bandung         │ 1,800,000,000   │1,600,000│ 89%│5678│
│  3 │ UGM                  │ Yogyakarta      │ 1,900,000,000   │1,700,000│ 89%│9012│
│ ...│ ...                  │ ...             │ ...             │ ...     │ ...│ ...│
│ 50 │ Univ. Negeri Papua   │ Jayapura        │   800,000,000   │  400,000│ 50%│3456│
├────┼──────────────────────┼─────────────────┼─────────────────┼─────────┼────┼────┤
│TOTL│ 50 Universitas       │ -               │ 150,000,000,000 │100,000  │ 67%│ -  │
└────┴──────────────────────┴─────────────────┴─────────────────┴─────────┴────┴────┘
```

**Columns**:
1. **No**: Auto-increment row number
2. **Nama Universitas**: Text (editable)
3. **Kab/Kota**: Dropdown (linked to kabupaten_kota table)
4. **Nominal**: Number (editable, format: Rupiah)
5. **Realisasi**: Number (editable, format: Rupiah)
6. **%**: Calculated field `= (Realisasi / Nominal) * 100`
7. **NPSN**: Text (Nomor Pokok Sekolah Nasional - unique identifier)

**Features**:
- **Cascading Filters**: Pilih Provinsi → Auto-filter Kabupaten/Kota
- **Inline Edit**: Click cell → Edit → Auto-save
- **Data Validation**: Realisasi <= Nominal
- **Conditional Formatting**: Color-coded % column
- **Export**: Download as Excel dengan semua formula preserved

**Database**: `institusi_pendidikan` table
```sql
CREATE TABLE institusi_pendidikan (
  id UUID PRIMARY KEY,
  npsn VARCHAR(20) UNIQUE,
  nama_institusi VARCHAR(255),
  jenjang ENUM('UNIVERSITAS', 'SMA', 'SMP', 'SD', 'PAUD'),
  kabupaten_kota_id UUID REFERENCES kabupaten_kota(id),
  nominal_alokasi DECIMAL(15,2),
  realisasi_total DECIMAL(15,2),
  persentase_penyerapan DECIMAL(5,2) GENERATED ALWAYS AS (
    (realisasi_total / nominal_alokasi) * 100
  ) STORED,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```



**URL**: `/dashboard/jenjang/sma`

**Layout**: Sama seperti Universitas, tapi filter `jenjang = 'SMA'`

**Example Data**:
```
┌────┬──────────────────────┬─────────────────┬─────────────────┬─────────┬────┤
│ No │ Nama SMA             │ Kab/Kota        │ Nominal         │ Realisasi│ %  │
├────┼──────────────────────┼─────────────────┼─────────────────┼─────────┼────┤
│  1 │ SMAN 1 Jakarta       │ Jakarta Pusat   │   800,000,000   │ 720,000 │ 90%│
│  2 │ SMAN 3 Bandung       │ Bandung         │   750,000,000   │ 675,000 │ 90%│
│ ...│ ...                  │ ...             │ ...             │ ...     │ ...│
│5000│ SMAN 1 Merauke       │ Merauke         │   500,000,000   │ 250,000 │ 50%│
└────┴──────────────────────┴─────────────────┴─────────────────┴─────────┴────┘
```

**Features**: Identik dengan Universitas



**URL**: `/dashboard/jenjang/paud`

**Layout**: Sama, filter `jenjang = 'PAUD'`

**Estimated Records**: ~80,000 PAUD di Indonesia



**Spreadsheet Layout**:
```
┌───────────────────────────────────────────────────────────────────────────────┐
│  USER MANAGER                                            [+ Add User] [⬇️ Export]│
├────┬──────────────────┬─────────────────────┬──────────────┬────────┬─────────┤
│ No │ Username         │ Email               │ Role         │ Status │ Actions │
├────┼──────────────────┼─────────────────────┼──────────────┼────────┼─────────┤
│  1 │ admin.kementerian│ admin@kemdikbud.id  │ ADMIN        │ Active │ [Edit]  │
│  2 │ viewer.jabar     │ viewer@jabar.go.id  │ VIEWER       │ Active │ [Edit]  │
│  3 │ admin.bandung    │ admin@bandung.go.id │ ADMIN_KABKOTA│ Active │ [Edit]  │
│ ...│ ...              │ ...                 │ ...          │ ...    │ ...     │
│ 50 │ viewer.audit     │ audit@bpk.go.id     │ AUDITOR      │ Active │ [Edit]  │
└────┴──────────────────┴─────────────────────┴──────────────┴────────┴─────────┘
```

**Features**:
1. **Add User**: Form popup untuk create user baru
2. **Edit**: Inline edit atau modal popup
3. **Delete**: Soft delete (set status = 'Inactive')
4. **Role Management**: Dropdown untuk set role
5. **Password Reset**: Button untuk reset password user

**User Roles**:
- `SUPER_ADMIN`: Full access semua menu
- `ADMIN`: Create, Read, Update data (tidak bisa delete)
- `VIEWER`: Read-only access
- `ADMIN_PROVINSI`: Admin untuk provinsi tertentu
- `ADMIN_KABKOTA`: Admin untuk kabupaten/kota tertentu
- `AUDITOR`: Read-only access + export data

**Database**: `users` table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  username VARCHAR(50) UNIQUE,
  email VARCHAR(255) UNIQUE,
  password_hash VARCHAR(255),
  role ENUM('SUPER_ADMIN', 'ADMIN', 'VIEWER', 'ADMIN_PROVINSI', 'ADMIN_KABKOTA', 'AUDITOR'),
  provinsi_id UUID REFERENCES provinsi(id) NULL,
  kabupaten_kota_id UUID REFERENCES kabupaten_kota(id) NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 3. DATABASE SCHEMA (SIMPLIFIED)

### 3.1 Core Tables

#### Table: tahun_anggaran
```sql
CREATE TABLE tahun_anggaran (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tahun INT UNIQUE NOT NULL,
  total_anggaran DECIMAL(15,2) NOT NULL,
  status VARCHAR(20) DEFAULT 'ACTIVE',
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### Table: provinsi
```sql
CREATE TABLE provinsi (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  kode_provinsi VARCHAR(10) UNIQUE,
  nama_provinsi VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### Table: alokasi_provinsi
```sql
CREATE TABLE alokasi_provinsi (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tahun_anggaran_id UUID REFERENCES tahun_anggaran(id),
  provinsi_id UUID REFERENCES provinsi(id),
  nominal_alokasi DECIMAL(15,2) NOT NULL,
  realisasi_total DECIMAL(15,2) DEFAULT 0,
  selisih DECIMAL(15,2) GENERATED ALWAYS AS (nominal_alokasi - realisasi_total) STORED,
  persentase_penyerapan DECIMAL(5,2) GENERATED ALWAYS AS (
    CASE WHEN nominal_alokasi > 0 
    THEN (realisasi_total / nominal_alokasi) * 100 
    ELSE 0 END
  ) STORED,
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Table: kabupaten_kota
```sql
CREATE TABLE kabupaten_kota (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  provinsi_id UUID REFERENCES provinsi(id),
  kode_kabupaten_kota VARCHAR(10) UNIQUE,
  nama_kabupaten_kota VARCHAR(255) NOT NULL,
  tipe ENUM('KABUPATEN', 'KOTA'),
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### Table: alokasi_kabupaten_kota
```sql
CREATE TABLE alokasi_kabupaten_kota (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  alokasi_provinsi_id UUID REFERENCES alokasi_provinsi(id),
  kabupaten_kota_id UUID REFERENCES kabupaten_kota(id),
  nominal_alokasi DECIMAL(15,2) NOT NULL,
  realisasi_total DECIMAL(15,2) DEFAULT 0,
  selisih DECIMAL(15,2) GENERATED ALWAYS AS (nominal_alokasi - realisasi_total) STORED,
  persentase_penyerapan DECIMAL(5,2) GENERATED ALWAYS AS (
    CASE WHEN nominal_alokasi > 0 
    THEN (realisasi_total / nominal_alokasi) * 100 
    ELSE 0 END
  ) STORED,
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Table: institusi_pendidikan (Universitas, SMA, SMP, SD, PAUD)
```sql
CREATE TABLE institusi_pendidikan (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  npsn VARCHAR(20) UNIQUE NOT NULL,
  nama_institusi VARCHAR(255) NOT NULL,
  jenjang ENUM('UNIVERSITAS', 'SMA', 'SMP', 'SD', 'PAUD') NOT NULL,
  kabupaten_kota_id UUID REFERENCES kabupaten_kota(id),
  nominal_alokasi DECIMAL(15,2) NOT NULL,
  realisasi_total DECIMAL(15,2) DEFAULT 0,
  selisih DECIMAL(15,2) GENERATED ALWAYS AS (nominal_alokasi - realisasi_total) STORED,
  persentase_penyerapan DECIMAL(5,2) GENERATED ALWAYS AS (
    CASE WHEN nominal_alokasi > 0 
    THEN (realisasi_total / nominal_alokasi) * 100 
    ELSE 0 END
  ) STORED,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### Table: users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  role ENUM('SUPER_ADMIN', 'ADMIN', 'VIEWER', 'ADMIN_PROVINSI', 'ADMIN_KABKOTA', 'AUDITOR') NOT NULL,
  provinsi_id UUID REFERENCES provinsi(id) NULL,
  kabupaten_kota_id UUID REFERENCES kabupaten_kota(id) NULL,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### 3.2 Database Triggers (Auto-Update Parent)

#### Trigger: Update Provinsi Realisasi ketika Kabupaten/Kota berubah
```sql
CREATE OR REPLACE FUNCTION update_provinsi_realisasi()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE alokasi_provinsi
  SET realisasi_total = (
    SELECT COALESCE(SUM(realisasi_total), 0)
    FROM alokasi_kabupaten_kota
    WHERE alokasi_provinsi_id = NEW.alokasi_provinsi_id
  )
  WHERE id = NEW.alokasi_provinsi_id;
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_provinsi_realisasi
AFTER INSERT OR UPDATE OF realisasi_total ON alokasi_kabupaten_kota
FOR EACH ROW
EXECUTE FUNCTION update_provinsi_realisasi();
```

#### Trigger: Update Kabupaten/Kota Realisasi ketika Institusi berubah
```sql
CREATE OR REPLACE FUNCTION update_kabkota_realisasi()
RETURNS TRIGGER AS $$
DECLARE
  v_kabkota_id UUID;
  v_alokasi_kabkota_id UUID;
BEGIN
  -- Get kabupaten_kota_id dari institusi
  SELECT kabupaten_kota_id INTO v_kabkota_id
  FROM institusi_pendidikan
  WHERE id = NEW.id;
  
  -- Get alokasi_kabupaten_kota_id
  SELECT id INTO v_alokasi_kabkota_id
  FROM alokasi_kabupaten_kota
  WHERE kabupaten_kota_id = v_kabkota_id
  LIMIT 1;
  
  -- Update realisasi_total kabupaten/kota
  UPDATE alokasi_kabupaten_kota
  SET realisasi_total = (
    SELECT COALESCE(SUM(realisasi_total), 0)
    FROM institusi_pendidikan
    WHERE kabupaten_kota_id = v_kabkota_id
  )
  WHERE id = v_alokasi_kabkota_id;
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_kabkota_realisasi
AFTER INSERT OR UPDATE OF realisasi_total ON institusi_pendidikan
FOR EACH ROW
EXECUTE FUNCTION update_kabkota_realisasi();
```

---

## 4. TECH STACK

### 4.1 Frontend
- **Framework**: Next.js 14 (App Router)
- **UI Library**: React 18
- **Styling**: Tailwind CSS
- **Component Library**: shadcn/ui (specifically Table, Input, Select components)
- **Spreadsheet Library**: 
  - **TanStack Table v8** (for advanced table features)
  - **react-data-grid** (for Excel-like editing experience) - **RECOMMENDED**
  - Atau custom implementation dengan contentEditable
- **State Management**: Zustand (for global state, cell editing state)
- **Form Handling**: React Hook Form (for Add/Edit modals)
- **Validation**: Zod
- **Charts**: Recharts (untuk Dashboard overview)
- **Number Formatting**: numeral.js (untuk format Rupiah)

### 4.2 Backend
- **Framework**: Next.js 14 API Routes
- **ORM**: Drizzle ORM
- **Database**: PostgreSQL 15+
- **Authentication**: Better Auth v2
- **Validation**: Zod
- **Excel Export**: ExcelJS (untuk export .xlsx dengan formula)

### 4.3 Database
- **PostgreSQL 15+** dengan generated columns untuk auto-calculation
- **Triggers** untuk cascade update (update parent saat child berubah)
- **Indexes** untuk performance (pada kolom yang sering di-query)

---

## 5. KEY FEATURES

### 5.1 Spreadsheet-Like Editing

**Inline Cell Editing**:
```typescript
// Example dengan react-data-grid
import DataGrid from 'react-data-grid';

const columns = [
  { key: 'nama_provinsi', name: 'Nama Provinsi', editable: false },
  { 
    key: 'nominal_alokasi', 
    name: 'Nominal', 
    editable: true,
    formatter: ({ row }) => formatRupiah(row.nominal_alokasi),
    editor: NumberEditor // Custom editor untuk number input
  },
  { 
    key: 'realisasi_total', 
    name: 'Realisasi', 
    editable: true,
    formatter: ({ row }) => formatRupiah(row.realisasi_total),
    editor: NumberEditor
  },
  { 
    key: 'persentase_penyerapan', 
    name: '%', 
    editable: false,
    formatter: ({ row }) => `${row.persentase_penyerapan.toFixed(2)}%`,
    cellClass: (row) => getPercentageColorClass(row.persentase_penyerapan)
  }
];

const handleRowsChange = async (rows, { indexes, column }) => {
  const updatedRow = rows[indexes[0]];
  
  // Auto-save to backend
  await updateProvinsi(updatedRow.id, {
    [column.key]: updatedRow[column.key]
  });
  
  // Update local state
  setRows(rows);
};

<DataGrid
  columns={columns}
  rows={rows}
  onRowsChange={handleRowsChange}
  className="rdg-light" // Styling
/>
```

### 5.2 Real-Time Calculation

**Frontend (Immediate Feedback)**:
```typescript
const calculateRow = (row) => ({
  ...row,
  selisih: row.nominal_alokasi - row.realisasi_total,
  persentase_penyerapan: (row.realisasi_total / row.nominal_alokasi) * 100
});

// Saat user edit cell
const handleCellEdit = (rowIndex, columnKey, value) => {
  const updatedRows = [...rows];
  updatedRows[rowIndex][columnKey] = value;
  
  // Recalculate
  updatedRows[rowIndex] = calculateRow(updatedRows[rowIndex]);
  
  // Recalculate TOTAL row
  const totalRow = {
    nama_provinsi: 'TOTAL',
    nominal_alokasi: sumBy(updatedRows, 'nominal_alokasi'),
    realisasi_total: sumBy(updatedRows, 'realisasi_total'),
    selisih: sumBy(updatedRows, 'selisih'),
    persentase_penyerapan: 
      (sumBy(updatedRows, 'realisasi_total') / sumBy(updatedRows, 'nominal_alokasi')) * 100
  };
  
  setRows([...updatedRows, totalRow]);
  
  // Async save to backend
  debouncedSave(updatedRows[rowIndex]);
};
```

**Backend (Database Triggers)**:
- PostgreSQL generated columns untuk auto-calculate `selisih` dan `persentase_penyerapan`
- Triggers untuk cascade update ke parent (Provinsi ← Kabupaten/Kota ← Institusi)

### 5.3 Conditional Formatting

```typescript
const getPercentageColorClass = (percentage) => {
  if (percentage >= 80) return 'bg-green-100 text-green-800';
  if (percentage >= 50) return 'bg-yellow-100 text-yellow-800';
  return 'bg-red-100 text-red-800';
};

// Apply to cell
<div className={getPercentageColorClass(row.persentase_penyerapan)}>
  {row.persentase_penyerapan.toFixed(2)}%
</div>
```

### 5.4 Excel Export dengan Formula

```typescript
import ExcelJS from 'exceljs';

const exportToExcel = async (data) => {
  const workbook = new ExcelJS.Workbook();
  const worksheet = workbook.addWorksheet('Provinsi');
  
  // Headers
  worksheet.columns = [
    { header: 'No', key: 'no', width: 5 },
    { header: 'Nama Provinsi', key: 'nama_provinsi', width: 20 },
    { header: 'Nominal', key: 'nominal_alokasi', width: 20 },
    { header: 'Realisasi', key: 'realisasi_total', width: 20 },
    { header: 'Selisih', key: 'selisih', width: 20 },
    { header: '%', key: 'persentase', width: 10 }
  ];
  
  // Data rows dengan formula
  data.forEach((row, index) => {
    const rowIndex = index + 2; // Start from row 2 (row 1 is header)
    worksheet.addRow({
      no: index + 1,
      nama_provinsi: row.nama_provinsi,
      nominal_alokasi: row.nominal_alokasi,
      realisasi_total: row.realisasi_total
    });
    
    // Add formula untuk Selisih dan %
    worksheet.getCell(`E${rowIndex}`).value = { 
      formula: `C${rowIndex}-D${rowIndex}` 
    };
    worksheet.getCell(`F${rowIndex}`).value = { 
      formula: `D${rowIndex}/C${rowIndex}*100` 
    };
  });
  
  // TOTAL row dengan SUM formula
  const totalRowIndex = data.length + 2;
  worksheet.addRow({
    no: '',
    nama_provinsi: 'TOTAL'
  });
  worksheet.getCell(`C${totalRowIndex}`).value = { 
    formula: `SUM(C2:C${totalRowIndex - 1})` 
  };
  worksheet.getCell(`D${totalRowIndex}`).value = { 
    formula: `SUM(D2:D${totalRowIndex - 1})` 
  };
  worksheet.getCell(`E${totalRowIndex}`).value = { 
    formula: `SUM(E2:E${totalRowIndex - 1})` 
  };
  worksheet.getCell(`F${totalRowIndex}`).value = { 
    formula: `D${totalRowIndex}/C${totalRowIndex}*100` 
  };
  
  // Styling
  worksheet.getRow(1).font = { bold: true };
  worksheet.getRow(totalRowIndex).font = { bold: true };
  
  // Number formatting
  worksheet.getColumn('nominal_alokasi').numFmt = '#,##0';
  worksheet.getColumn('realisasi_total').numFmt = '#,##0';
  worksheet.getColumn('selisih').numFmt = '#,##0';
  worksheet.getColumn('persentase').numFmt = '0.00%';
  
  // Download
  const buffer = await workbook.xlsx.writeBuffer();
  const blob = new Blob([buffer], { 
    type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' 
  });
  saveAs(blob, 'Provinsi_Data.xlsx');
};
```

### 5.5 Filter & Search

```typescript
// Filter by Provinsi (untuk menu Kabupaten/Kota)
const [selectedProvinsi, setSelectedProvinsi] = useState(null);
const [searchTerm, setSearchTerm] = useState('');

const filteredData = useMemo(() => {
  let result = data;
  
  // Filter by Provinsi
  if (selectedProvinsi) {
    result = result.filter(row => 
      row.provinsi_id === selectedProvinsi
    );
  }
  
  // Search by name
  if (searchTerm) {
    result = result.filter(row =>
      row.nama_kabupaten_kota.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
  
  return result;
}, [data, selectedProvinsi, searchTerm]);
```

---

## 6. API ENDPOINTS (SIMPLIFIED)

### 6.1 Dashboard
- `GET /api/dashboard/summary?tahun={tahun}` - Get dashboard overview

### 6.2 Provinsi
- `GET /api/provinsi?tahun={tahun}` - List all provinsi dengan alokasi
- `PUT /api/provinsi/:id` - Update nominal atau realisasi
- `GET /api/provinsi/:id` - Get detail provinsi

### 6.3 Kabupaten/Kota
- `GET /api/kabupaten-kota?provinsi_id={id}&tahun={tahun}` - List kabupaten/kota
- `PUT /api/kabupaten-kota/:id` - Update nominal atau realisasi
- `POST /api/kabupaten-kota` - Create new kabupaten/kota
- `DELETE /api/kabupaten-kota/:id` - Delete kabupaten/kota

### 6.4 Jenjang Pendidikan
- `GET /api/institusi?jenjang={jenjang}&provinsi_id={id}&kabupaten_kota_id={id}` - List institusi
- `PUT /api/institusi/:id` - Update nominal atau realisasi
- `POST /api/institusi` - Create new institusi
- `DELETE /api/institusi/:id` - Delete institusi
- `POST /api/institusi/bulk-import` - Import from Excel

### 6.5 Users
- `GET /api/users` - List all users
- `POST /api/users` - Create new user
- `PUT /api/users/:id` - Update user
- `DELETE /api/users/:id` - Soft delete user

### 6.6 Export
- `GET /api/export/provinsi?tahun={tahun}&format=xlsx` - Export provinsi to Excel
- `GET /api/export/kabupaten-kota?provinsi_id={id}&format=xlsx` - Export kabkota to Excel
- `GET /api/export/institusi?jenjang={jenjang}&format=xlsx` - Export institusi to Excel

---

## 7. MVP ROADMAP (REVISED)

### Sprint 1 (Week 1-2): Foundation
**Goal**: Setup project, database, authentication, dan basic dashboard

**Tasks**:
1. Initialize Next.js 14 project
2. Setup Drizzle ORM + PostgreSQL
3. Create database schema (6 tables: tahun_anggaran, provinsi, alokasi_provinsi, kabupaten_kota, alokasi_kabupaten_kota, institusi_pendidikan, users)
4. Seed data: 38 provinsi, sample kabupaten/kota, sample institusi
5. Setup authentication (Better Auth)
6. Create main dashboard layout dengan sidebar menu
7. Implement Dashboard page (overview with cards & chart)

**Deliverable**: Dashboard homepage working, user can login

---

### Sprint 2 (Week 3-4): Spreadsheet Interface - Provinsi & Kabupaten/Kota
**Goal**: Implement spreadsheet-like interface untuk Provinsi dan Kabupaten/Kota

**Tasks**:
1. Install react-data-grid atau TanStack Table
2. Create Provinsi page dengan:
   - Spreadsheet table (editable cells)
   - Inline editing dengan auto-save
   - Conditional formatting (color-coded %)
   - TOTAL row dengan SUM formula
   - Export to Excel button
3. Create Kabupaten/Kota page dengan:
   - Filter by Provinsi dropdown
   - Same spreadsheet features as Provinsi
4. Implement database triggers untuk cascade update
5. Create API endpoints untuk CRUD operations
6. Test real-time calculation (edit cell → auto-update parent)

**Deliverable**: Provinsi & Kabupaten/Kota menu functional dengan spreadsheet editing

---

### Sprint 3 (Week 5-6): Jenjang Pendidikan (5 Sub-Menus)
**Goal**: Implement 5 sub-menus untuk jenjang pendidikan dengan filtering

**Tasks**:
1. Create shared Institusi component (reusable untuk 5 jenjang)
2. Implement filtering:
   - Filter by Jenjang (Universitas/SMA/SMP/SD/PAUD)
   - Filter by Provinsi (dropdown)
   - Filter by Kabupaten/Kota (cascading dropdown)
   - Search by Nama Institusi
3. Create 5 routes:
   - `/dashboard/jenjang/universitas`
   - `/dashboard/jenjang/sma`
   - `/dashboard/jenjang/smp`
   - `/dashboard/jenjang/sd`
   - `/dashboard/jenjang/paud`
4. Implement pagination (handle 150,000+ SD records)
5. Add bulk import from Excel (untuk mass data entry)
6. Test cascade update: Edit Institusi → Update Kabkota → Update Provinsi

**Deliverable**: All 5 jenjang pendidikan menus functional dengan filtering & pagination

---

### Sprint 4 (Week 7-8): User Management & Final Polish
**Goal**: User management, export functionality, dan final testing

**Tasks**:
1. Create User Manager page:
   - CRUD users
   - Role assignment
   - Password reset
2. Implement role-based access control:
   - SUPER_ADMIN: Full access
   - ADMIN: Can edit data
   - VIEWER: Read-only
   - ADMIN_PROVINSI: Limited to their provinsi
   - ADMIN_KABKOTA: Limited to their kabkota
3. Enhanced export functionality:
   - Export with formula preserved
   - Export with conditional formatting
   - Export filtered data
4. Performance optimization:
   - Database indexing
   - Query optimization
   - Lazy loading untuk large datasets
5. Final testing & bug fixes
6. Documentation (user manual, admin guide)

**Deliverable**: Production-ready application dengan user management & export features

---

## 8. SUCCESS METRICS

### 8.1 Performance
- Page load time: < 1 second (dashboard, provinsi, kabkota)
- Large table load time: < 2 seconds (SD dengan 150,000 records)
- Cell edit → Save to DB: < 500ms
- Cascade update: < 1 second (update institusi → propagate to provinsi)

### 8.2 User Experience
- Spreadsheet editing feels like Excel (no lag, smooth scrolling)
- Conditional formatting updates in real-time
- Export preserves all formulas and formatting
- Filter/search returns results in < 500ms

### 8.3 Data Integrity
- 100% accuracy in cascade calculations
- No data loss during inline editing
- Validation prevents invalid data entry (realisasi > nominal)

---

## 9. TECH RECOMMENDATIONS

### Spreadsheet Library Comparison

| Library | Pros | Cons | Recommendation |
|---------|------|------|----------------|
| **react-data-grid** | Excel-like experience, built-in inline editing, performance-optimized | Steeper learning curve | ⭐ **BEST for this project** |
| **TanStack Table v8** | Highly customizable, hooks-based, lightweight | Need custom editor implementation | Good alternative |
| **AG Grid** | Feature-rich, enterprise-grade | Heavy, expensive for commercial use | Overkill untuk project ini |
| **Custom with contentEditable** | Full control | Time-consuming to build | Not recommended |

**Final Recommendation**: **react-data-grid** karena:
- Built-in inline editing
- Excel-like keyboard navigation
- Performance dengan virtualization (handle 100k+ rows)
- Open-source & free

---

## 10. DEPLOYMENT PLAN

### 10.1 Development
- **Frontend + API**: Local development (localhost:3000)
- **Database**: Railway PostgreSQL (development instance)

### 10.2 Production
- **Frontend + API**: Vercel (auto-scaling serverless)
- **Database**: AWS RDS PostgreSQL (production-grade)
- **File Storage**: AWS S3 (untuk uploaded files, exports)
- **Domain**: Custom domain dengan SSL (Cloudflare DNS)

---

**Document Status**: ✅ REVISED & APPROVED  
**Version**: 2.0  
**Last Updated**: April 13, 2026  
**Next Review**: After Sprint 1 Completion
