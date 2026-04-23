# SBOM Dashboard

Static HTML dashboard untuk memvisualisasikan hasil scan **Trivy** dalam format **CycloneDX 1.6**. Tidak memerlukan server, framework, atau instalasi apapun — cukup buka file di browser.

---

## Screenshot

```
┌─────────────────────────────────────────────────────────┐
│ SBOMviewer  CycloneDX 1.6              [Load JSON]      │
├─────────────────────────────────────────────────────────┤
│ BOM METADATA                                            │
│  Target · Version · Timestamp · Scanner · SerialNumber  │
├──────────┬──────────┬──────────┬──────────┬────────────┤
│ 312      │  3 CRIT  │  12 HIGH │  28 MED  │  41 LOW   │
│ Components│         │          │          │            │
├─────────────────────────────────────────────────────────┤
│ Components │ Vulnerabilities │ Dependencies             │
│ ─────────────────────────────────────────────────────── │
│ [Search…] [Type ▾]                     312 / 312        │
│  Name          Version   Type   PURL   Licenses  Hash  │
│  ▸ lodash      4.17.15   lib    pkg:…  MIT             │
│  ▸ openssl     3.0.2     lib    pkg:…  Apache-2.0      │
└─────────────────────────────────────────────────────────┘
```

---

## Cara Pakai

### 1. Generate SBOM dengan Trivy

**Scan container image:**
```bash
trivy image --format cyclonedx --output sbom.json nginx:latest
```

**Scan filesystem / source code:**
```bash
trivy fs --format cyclonedx --output sbom.json ./my-project
```

**Scan dengan vuln + license sekaligus:**
```bash
trivy image \
  --format cyclonedx \
  --scanners vuln,license \
  --output sbom.json \
  alpine:3.18
```

**Scan Kubernetes cluster:**
```bash
trivy k8s --format cyclonedx --output sbom.json cluster
```

### 2. Buka Dashboard

1. Buka file `sbom-dashboard.html` di browser (Chrome, Firefox, Edge, Safari)
2. Drag & drop file `sbom.json` ke area drop zone, **atau** klik tombol **Load JSON**
3. Dashboard langsung render — tidak ada loading ke server

> **Privacy:** Semua data diproses 100% di browser (in-memory JavaScript). File JSON tidak pernah dikirim ke mana pun.

---

## Fitur

### BOM Metadata
Ringkasan informasi BOM di bagian atas:
- Nama target dan versi
- Timestamp scan
- Nama & versi scanner (Trivy)
- Spec version dan Serial Number

### Summary Cards
6 kartu ringkasan di bawah metadata:
| Card | Keterangan |
|---|---|
| Components | Total komponen yang ditemukan |
| Critical | Jumlah vuln severity Critical |
| High | Jumlah vuln severity High |
| Medium | Jumlah vuln severity Medium |
| Low | Jumlah vuln severity Low |
| Info / None | Vuln severity Info, None, atau Unknown |

### Tab: Components

Tabel semua komponen dalam SBOM dengan kolom:
- **Name** — nama package/library
- **Version** — versi yang terdeteksi
- **Type** — tipe komponen (`library`, `container`, `application`, dll)
- **PURL** — Package URL (identifier unik, bisa di-hover untuk full URL)
- **Licenses** — lisensi yang terdeksi (MIT, Apache-2.0, GPL, dll)
- **Hashes** — hash integritas komponen (SHA-256, SHA-1, MD5)

**Expand row** (klik baris) untuk detail tambahan: BOM-ref, PURL penuh, CPE, deskripsi, semua hash, supplier, author.

Filter tersedia:
- Search bar: cari berdasarkan nama, versi, atau PURL
- Dropdown **Type**: filter berdasarkan tipe komponen
- Klik header kolom untuk sort ascending/descending

### Tab: Vulnerabilities

Tabel semua CVE/vulnerability yang ditemukan:
- **CVE / ID** — identifier kerentanan (CVE-XXXX-XXXXX)
- **Severity** — badge berwarna: Critical / High / Medium / Low / Info / None
- **Score** — nilai CVSS dengan visualisasi bar warna
- **Method** — metode scoring (CVSSv31, CVSSv2, OWASP)
- **Affected** — chip komponen yang terdampak
- **VEX State** — status exploitability jika ada analisis VEX

Filter tersedia:
- Search bar: cari berdasarkan CVE ID atau nama komponen
- Tombol severity filter multi-select (bisa kombinasi, misal High + Critical)
- Klik header untuk sort berdasarkan severity atau score

**Expand row** untuk detail: deskripsi lengkap, source URL, CWE list, semua rating CVSS, CVSS vector string, VEX justification dan analysis detail.

### Tab: Dependencies

Dependency graph dalam format flat list:
- Menampilkan setiap komponen beserta daftar `dependsOn`-nya
- Chip-chip menunjukkan referensi ke komponen lain
- Cocok untuk analisis transitive dependency

---

## Format yang Didukung

| Field CycloneDX | Didukung |
|---|---|
| `metadata.component` | ✅ |
| `metadata.tools` | ✅ |
| `metadata.timestamp` | ✅ |
| `components[].name/version/type` | ✅ |
| `components[].purl` | ✅ |
| `components[].cpe` | ✅ |
| `components[].licenses` | ✅ |
| `components[].hashes` | ✅ |
| `components[].supplier/author` | ✅ |
| `vulnerabilities[].id` | ✅ |
| `vulnerabilities[].ratings` (CVSS score, severity, vector) | ✅ |
| `vulnerabilities[].cwes` | ✅ |
| `vulnerabilities[].affects` | ✅ |
| `vulnerabilities[].analysis` (VEX state, justification, detail) | ✅ |
| `vulnerabilities[].source` | ✅ |
| `dependencies[]` | ✅ |

> Tool ini dirancang untuk output Trivy, tetapi kompatibel dengan SBOM CycloneDX 1.6 dari tool lain (Syft, cdxgen, dll) selama mengikuti skema yang sama.

---

## Tidak Memerlukan

- ❌ Node.js / npm
- ❌ Python / pip
- ❌ Web server
- ❌ Internet (setelah file di-load, Google Fonts opsional)
- ❌ Login / API key

---

## Referensi

- [Trivy Documentation](https://trivy.dev/latest/docs/)
- [CycloneDX Specification 1.6](https://cyclonedx.org/specification/overview/)
- [CycloneDX JSON Schema 1.6](https://cyclonedx.org/schema/bom-1.6.schema.json)
- [VEX — Vulnerability Exploitability eXchange](https://cyclonedx.org/capabilities/vex/)
