# AI Workflows & Skills Repository

Repositori ini menyimpan kumpulan instruksi operasional global, pedoman (guidelines), dan spesifikasi *skill* atau alur kerja (workflow) untuk eksekusi agen AI. Lingkungan ini dirancang untuk memastikan AI mematuhi standar *clean architecture*, skalabilitas, dan keamanan ketika melakukan berbagai tugas perangkat lunak seperti *game development* atau iterasi pembuatan *skill* baru.

## Fitur Utama

- **Global Execution Standards (`global.md`)**: Berisi aturan operasional inti, batasan keamanan, standar *coding* (Golang, TypeScript, HTML/CSS), kebijakan *database*, dan instruksi *bug hunting* yang wajib diikuti oleh agen AI di setiap eksekusi.
- **Game Development Skill (`game-dev.md`)**: Pedoman bagi AI untuk membangun *browser games* dan komponen interaktif menggunakan HTML5 Canvas, WebGL, dan React (beserta implementasi fisika, deteksi benturan, sistem *scoring*, dan siklus *game loop*).
- **Skill Creator & Evaluator (`skill-creator.md`)**: Sebuah *meta-skill* yang memungkinkan AI untuk merancang, membuat, menguji, dan mengevaluasi *skill* baru melalui serangkaian iterasi dengan umpan balik dari pengguna. Termasuk pembuatan kasus uji, perbandingan *baseline*, dan optimasi deskripsi (*trigger evaluation*).
- **Quantitative & Qualitative Benchmarking**: Memanfaatkan serangkaian *subagent* dan *script* Python untuk mengukur performa *skill* AI secara terukur dan visual.

## Arsitektur dan Alur Eksekusi (Architecture & Execution Flow)

Sistem *skill* menggunakan model hierarki *progressive disclosure* untuk menghindari *context window bloat*:
1. **Metadata Level**: Berisi nama dan deskripsi pemicu (*trigger description*).
2. **SKILL.md Body**: Instruksi utama dan alur kerja (contoh: `game-dev.md`, `skill-creator.md`).
3. **Bundled Resources**: Referensi eksternal, *template*, dan *script* eksekusi yang hanya dimuat saat dibutuhkan.

Alur eksekusi pembuatan dan pengujian *skill* (*Skill Creation Loop*):
- **Preparation**: Menentukan tujuan *skill* dan merancang *draft* awal.
- **Subagent Parallel Execution**: Menjalankan *test case* secara bersamaan (satu sub-agen menggunakan *skill*, satu lagi menggunakan *baseline*).
- **Grading & Aggregation**: Sub-agen `grader.md` menilai *output* berdasarkan asersi kualitatif/kuantitatif, sementara skrip agregasi `aggregate_benchmark.py` mengumpulkan data performa (token, waktu).
- **Review**: Menggunakan antarmuka visual (`eval-viewer`) untuk pengguna meninjau dan memberikan masukan umpan balik (*feedback*).
- **Iteration**: Menyempurnakan *skill* berdasarkan masukan dan mengulangi siklus.

## Struktur Direktori (Folder Structure)

```text
ai-workflows/
├── agents/             # Instruksi prompt khusus untuk sub-agen pendukung evaluasi
│   ├── analyzer.md     # Sub-agen untuk menganalisis hasil benchmark
│   ├── comparator.md   # Sub-agen untuk perbandingan blind A/B
│   └── grader.md       # Sub-agen untuk menilai output terhadap asersi
├── assets/             # Berkas pendukung seperti template antarmuka
│   └── eval_review.html
├── eval-viewer/        # Skrip dan template HTML untuk meninjau hasil evaluasi secara visual
│   ├── generate_review.py
│   └── viewer.html
├── references/         # Dokumentasi spesifik atau skema teknis pendukung
│   ├── patterns.md
│   ├── schemas.md
│   └── webgl.md
├── scripts/            # Kumpulan utilitas Python untuk proses benchmark dan evaluasi
│   ├── aggregate_benchmark.py
│   ├── generate_report.py
│   ├── improve_description.py
│   ├── package_skill.py
│   ├── quick_validate.py
│   ├── run_eval.py
│   └── run_loop.py
├── game-dev.md         # Instruksi spesifik untuk pembuatan game HTML5 & React
├── global.md           # Aturan inti (Core Operating Directives) agen AI
└── skill-creator.md    # Instruksi meta-skill untuk menciptakan skill AI baru
```

## Teknologi & Dependensi (Technology Stack)

- **AI Prompting**: Format Markdown (`.md`) dengan dukungan YAML *frontmatter* untuk metadata *skill*.
- **Evaluation Scripts**: Python 3.x (dengan kapabilitas *subprocess* dan agregasi JSON).
- **Evaluation Viewer**: HTML5 murni, CSS, dan vanilla JavaScript untuk *rendering* hasil.

## Pemasangan dan Pengembangan (Installation & Development Setup)

Repositori ini berjalan secara natural pada platform atau terminal asisten AI yang mendukung pemuatan *workflow* dan *subagent* (seperti Claude Code atau Gemini CLI).

Untuk mengemas (*package*) sebuah *skill* hasil pengembangan:
```bash
python -m scripts.package_skill /path/to/skill-folder
```
Ini akan menghasilkan berkas `.skill` yang siap diimpor.

## Pengujian dan Eksekusi (Testing Strategy & Execution)

Setiap *skill* harus divalidasi melalui sistem asersi. Semua hasil evaluasi *test case* disarankan disimpan dalam format JSON mengikuti struktur `references/schemas.md`. 
Untuk melakukan evaluasi deskripsi *trigger*:
```bash
python -m scripts.run_loop \
  --eval-set path-to-trigger-eval.json \
  --skill-path path-to-skill \
  --model <model-id> \
  --max-iterations 5
```

## Keamanan dan Pertimbangan Infrastruktur (Security Considerations)

Berdasarkan aturan utama di `global.md`, eksekusi skrip dan asisten AI dilarang keras untuk:
- Membaca, mengekspos, atau mencetak data sensitif (`.env`, `credential files`, `private keys`).
- Melakukan mutasi *database* langsung tanpa *file migration*.
- Menjalankan skrip destruktif (seperti `rm -rf` atau *mass drops*).
Seluruh operasi sistem atau iterasi evaluasi yang dilakukan oleh agen AI dianggap beroperasi dalam skala *production-sensitive* yang memerlukan proteksi integritas tingkat tinggi.
