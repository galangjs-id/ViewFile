# ViewFile

Universal read-only file viewer. Buka file, lihat isinya — tanpa bisa edit.

## Stack
- Vite + React + TypeScript
- Tailwind CSS v4
- lucide-react (icons)
- highlight.js core (syntax highlighting — lazy loaded)
- pdfjs-dist (PDF rendering — lazy loaded)

## Jalankan
```
npm install
npm run dev
```

Build production: `npm run build` → output di `dist/`. Deploy ke Vercel:
import repo ini di vercel.com, auto-detect Vite, langsung Deploy.

## Status: SEMUA 10 STEP SELESAI ✅

1. Foundation + UI utama
2. File picker + drag & drop (`hooks/useFileInput.ts`)
3. File detection system (`utils/mime-types.ts`, `utils/file-detection.ts`)
4. TextViewer — .txt, .log, .csv
5. CodeViewer — .js, .ts, .jsx, .tsx, .css, .html, .xml, .yaml, .md
   (syntax highlight + line numbers + copy; .html/.htm dapat toggle
   Source/Preview dengan iframe sandbox penuh, JS gak dieksekusi)
6. JsonViewer — collapsible tree, fallback raw+error kalau JSON invalid
7. ImageViewer — png/jpg/jpeg/webp/gif/svg/bmp, zoom in/out/fit/100%
8. PdfViewer — multi-page, lazy render per halaman (IntersectionObserver),
   zoom, fit width, page navigation + indicator
9. UnsupportedViewer — pesan jelas, bukan viewer palsu
10. Polish responsive & performance:
    - CodeViewer & PdfViewer di-lazy-load (`React.lazy`) — highlight.js
      dan pdf.js (+worker, ~2.2 MB) cuma ke-load kalau file yang relevan
      dibuka. Bundle utama ~242 KB gzip ~76 KB.
    - Guard ukuran file: skip syntax highlighting di atas ~400 KB source,
      skip baca text di atas 25 MB, skip PDF di atas 150 MB — daripada
      nge-freeze browser.
    - PDF: cuma page yang mau kelihatan yang di-render ke canvas.
    - Toolbar per-viewer scroll horizontal di layar sempit; header
      truncate nama file panjang.

## Arsitektur
```
src/
  components/
    layout/Header.tsx
    file-picker/EmptyState.tsx
    viewer/
      ViewerRouter.tsx      <- pilih viewer berdasarkan file-detection
      TextViewer.tsx
      CodeViewer.tsx        <- lazy
      JsonViewer.tsx
      ImageViewer.tsx
      PdfViewer.tsx         <- lazy
      UnsupportedViewer.tsx
      shared/
        CodeBlockBase.tsx   <- gutter baris + <pre> dipakai Text & Code
        JsonNode.tsx         <- node tree JSON yang collapsible
        PdfPage.tsx          <- 1 halaman PDF, lazy render on-scroll
    ui/
      ViewfinderFrame.tsx    <- motif visual utama (bracket kamera)
      Toolbar.tsx
      StateMessage.tsx       <- Loading/Error/Unsupported
  hooks/
    useFileInput.ts          <- klik + drag&drop
    useFileContent.ts        <- baca file sesuai kategori (text/url/bytes)
    usePdfDocument.ts
    useCopyToClipboard.ts
  utils/
    mime-types.ts            <- tabel extension -> category/language/label
    file-detection.ts
    file-size.ts
    highlight.ts             <- wrapper highlight.js/core (lazy chunk)
    escape-html.ts           <- escape polos, sengaja dipisah biar TextViewer
                                 & JsonViewer gak ikut narik highlight.js
    pdf-setup.ts             <- konfigurasi worker pdf.js

Semua file diproses client-side — gak ada upload ke server manapun.
```

## Yang mungkin masih pengen dipoles nanti (opsional, di luar 10 step asli)
- Expand-all/collapse-all di JsonViewer
- Keyboard shortcuts (arrow key next/prev page di PDF, dsb.)
- Virtualisasi baris untuk file text/code yang jumlah barisnya sangat
  ekstrem (jutaan baris) — saat ini mengandalkan guard ukuran file
  (skip highlighting/baca di atas batas) daripada windowing baris
