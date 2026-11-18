# PANDUAN STANDAR LKPD DIGITAL — SMPN 2 KARAWANG BARAT
Versi dokumen: 1.0 • Acuan implementasi: MTK-003 (Tes Formatif) & MTK-004 (Pythagoras) — STATUS FIX

## 1. ARSITEKTUR
- Portal & server: Google Apps Script (GAS) Web App (index.html + fragmen: Stylesheet, LKPD, Periksa, REKAP, KOLAB, NILAI, PERFORMA).
- Halaman siswa: statis di GitHub Pages `wendirosandi.github.io/mtk/...` (folder `LKPD/` untuk file LKPD; TANPA folder per-LKPD).
- Database: Google Spreadsheet (sheet: Pengaturan, User Siswa, LinkLKPD, LinkTerpakai, RequestLink, Rekap, DaftarLKPD, KunciLKPD, PeriksaLKPD, NilaiLKPD, RequestKolab, RequestKategori, Performa, PengaturanDirect, RekapArsip, Rekap_backup).
- Enkripsi token link: AES (CryptoJS 3.1.9-1) payload 6 bagian: `kode|username|nama|kelas|kategori|gen`.
- Password: SHA-256 (migrasi otomatis dari plaintext saat login pertama).

## 2. SIKLUS STATUS LKPD (kolom Status, DaftarLKPD)
Aktif → generate link & request siswa. | Tidak Aktif → hanya LoginDirect. | Arsip → read-only, nilai tetap tampil. | Dihapus → soft-delete, sembunyi dari siswa.
Generate link HANYA untuk status Aktif (guard server).

## 3. KOLABORASI (kolom Kolaborasi, DaftarLKPD)
'' / Tidak → tanpa gate. | Ya → gate: [Kerjakan Sendiri (langsung)] atau [Berpasangan (menunggu persetujuan guru)]. | Wajib → gate: [Berpasangan AUTO-APPROVE] atau [Kerjakan Sendiri (menunggu persetujuan guru)].
- Pasangan difilter kategori: A↔{A,B}, B↔{A,B,C}, C↔{B,C}.
- Siswa berstatus "Siswa 2" dari pasangan disetujui → diblokir membuka link sendiri.
- Persetujuan manual guru (termasuk opsi SOLO) via panel 🤝 Kolaborasi.
- Saat kirim, payload membawa `username2`; server memverifikasi baris RequestKolab 'Disetujui' lalu menduplikasi baris Rekap identik untuk Siswa 2 (kolom E saling referensi nama pasangan).

## 4. SKEMA REKAP (18 KOLOM — JANGAN UBAH URUTAN)
1 KodeLKPD, 2 Nama1, 3 Kelas, 4 Kategori, 5 Nama2, 6 JawabanLKPD, 7 SkorLKPD, 8 WaktuPengerjaanLKPD, 9 WaktuKirimLKPD, 10 SkorKuis, 11 JawabanKuis, 12 WaktuKuis, 13 WaktuKirimKuis, 14 Percobaan, 15 Username, 16-18 Pelanggaran(Fullscreen/Tab/Copy).
Kunci baris = Username + KodeLKPD (percobaan baru = update baris yang sama).

## 5. PAYLOAD STANDAR (POST ke GAS, Content-Type text/plain)
LKPD: {tipe:'LKPD', kode, nama1, kelas, kategori, username, nama2, jawaban, waktuPengerjaan, percobaan, autoNilai, totalSoal, username2}
KUIS: {tipe:'KUIS', kode, nama1, kelas, kategori, username, skor, jawaban, waktuPengerjaan, percobaan, username2}
autoNilai → menulis kolom G Rekap + sheet NilaiLKPD. Skor kuis → kolom J.

## 6. KONVENSI WAJIB HALAMAN SISWA (statis)
1) Loader overlay transparan + teks aktivitas per tahap + toast umpan balik + jaring error global (error & unhandledrejection).
2) Identitas dual-mode: token `?d=` ATAU sesi LoginDirect (localStorage DIRECT_SESSION); tampilkan identitas minimal (Nama & Kelas) terkunci.
3) Salt deterministik (mulberry32) agar soal TIDAK berubah saat refresh; tombol "Acak Soal Baru" = salt baru.
4) Autosave semua input ke localStorage per username+kode+gen; pulihkan saat load.
5) PRINSIP SCAFFOLD: setiap angka pada langkah pengerjaan = INPUT siswa (termasuk salinan "Diketahui"); penilaian 1 poin per kotak.
6) Kunci per kegiatan/nomor (🔒) + modal peringatan sebelum kirim (daftar terkunci/belum/kosong; tombol: Kunci Semua & Kirim / Kirim Apa Adanya / Batal).
7) Umpan benar/salah per input saat evaluasi (kelas input-correct / input-wrong) + kartu skor + tombol kirim.
8) UA-gate CBT (`CBT-MTK-SMPN2Karbar`) hanya untuk halaman alur LoginDirect (LoginDirect.html & TesFormatif1.html).
9) Tombol 🚪 Keluar di layar sukses: hapus sesi lokal → kembali LoginDirect.

## 7. STANDAR GENERATOR — MTK-003 (TES FORMATIF, sesuai PDF spesifikasi)
10 soal; kesulitan dari kategori siswa (A=sulit, B=sedang, C=mudah).
1) Pangkat: (B1^e1×B2^e2)/((B3^e3)²); basis pangkat dari satu prima; jumlah NILAI basis berbeda: C=1, B=2, A=3. Kunci `p^N` (alt integer).
2) Sama, hasil negatif → kunci `1/p^M` (alt `p^-M`).
3) Sama, hasil nol → kunci `p^0` (alt `1`).
4) Akar=pangkat pecahan `ⁿ√(B^e)`; WAJIB e≥2 dan hasil pangkat ≥2 (tidak boleh pangkat 1); kunci integer.
5) Pecahan akar (a√x±b√y)/(c√z); jumlah akar perlu disederhanakan: C=1, B=2, A=3; kunci pecahan sederhana (alt desimal).
6) Hanya ×/÷ akar, hasil integer.
7) Ke bentuk baku n positif; angka penting: C=2, B=3, A=4; INPUT = kolom desimal + SLIDER pangkat (−15..15), tanpa toolbar.
8) Ke bentuk baku n negatif; kontrol sama dgn 7.
9) Dari bentuk baku n positif → angka; input polos (titik ribuan diterima).
10) Dari bentuk baku n negatif → angka; input polos (koma desimal diterima).
Normalisasi jawaban: samakan ×/x/*; strip titik ribuan; koma desimal→titik; bandingkan numerik (eps 1e-9) bila keduanya numerik, selain itu string + daftar alt.

## 8. STANDAR GENERATOR — MTK-004 (PYTHAGORAS)
LKPD: Kegiatan 1 (kapal/pulau, SVG kompas) & Kegiatan 2 (3 kasus jenis segitiga) — semua angka langkah = input.
Kuis 5 soal, 3 menit/soal (total 15 mnt), CEK terkunci hingga 1/3 waktu, TANPA retry (konfirmasi sebelum CEK), blueprint pola per kategori.
Q1 Benar/Salah 10 variasi rumus; huruf: C=a,b,c siku di C; B=siku acak; A=set huruf acak {p,q,r},{g,h,i},{x,y,z},{u,v,w}.
Q2 Menjodohkan jenis (Siku/Lancip/Tumpul/Bukan Segitiga); kasus: C=3, B=4, A=5; desimal & "Bukan Segitiga" porsi kecil di C, wajib ada di A.
Q3 Scaffold rumus kuadrat (cari c/a/b). Q4 Scaffold rumus akar. Q5 Gabungan: keluarga F1–F5:
 F1 rantai 2 segitiga (hip t1 = sisi siku t2): (3,4,5)+(5,12,13); (9,12,15)+(15,8,17); (12,16,20)+(20,21,29); (6,8,10)+(10,24,26); (15,20,25)+(25,60,65).
 F2 berbagi sisi siku vertikal: (5,12,13)+(12,35,37); (8,15,17)+(15,20,25); (7,24,25)+(24,32,40).
 F3 persegi+segitiga (s,x,h): (12,16,20); (9,12,15); (5,12,13); (15,20,25); (10,24,26).
 F4 layang-layang diagonal ⊥ (p; q1→s1 & q2→s2): (12;9→15&16→20); (8;6→10&15→17); (9;12→15&40→41); (16;12→20&30→34).
 F5 rantai 3 segitiga (sulit): (9,12,15)→(15,20,25)→(25,60,65).
Pemetaan: C=F3/F1 kecil; B=F1/F2; A=F4/F5/F1 besar. Semua triple menjamin integer tiap langkah.

## 9. ARSIP PDF
Halaman `MTK004_print.html`: 3 versi tetap (seed kunci, bukan acak) + halaman kunci hanya dengan `?kunci=1`. Cetak via browser → PDF. Prosedur sama untuk LKPD lain: buat `<KODE>_print.html` dengan tabel seed tetap per kategori.

## 10. CHECKLIST LKPD BARU (urutan kerja)
1) Tulis spesifikasi generator + scaffold (ikuti §7/§8). 2) Buat file statis di `/mtk/LKPD/<KODE>.html` memenuhi §6. 3) Tambah baris DaftarLKPD (FileHTML URL lengkap, Status, Kolaborasi). 4) Bila auto-graded: payload autoNilai; bila manual: isi KunciLKPD. 5) Uji: token, gate kolaborasi, salt/refresh, kunci, autosave, kirim (cek kolom G & J Rekap), pasangan (duplikasi baris). 6) Buat `<KODE>_print.html` + arsip PDF. 7) Update Rekap/Performa bila ada elemen kelengkapan baru.

## 11. PERFORMA & KELENGKAPAN
Sheet Performa (11 kolom): Username, Kode, PerformaDiri, PerformaLKPD, Sikap, Pengetahuan, Keterampilan, CatatanSiswa, CatatanGuru, UpdateSiswa, UpdateGuru. Skala: Kurang/Cukup/Baik/Sangat Baik. Siswa isi 2 pertama (setelah LKPD terkirim); guru isi 3 terakhir (auto-save).
Chip LENGKAP = 7 elemen: LKPD kirim + Kuis kirim + 5 performa; ikon ✅ vs ⏳ n/7 + popup daftar kekurangan; tampil di kartu Nilai siswa & tabel Performa guru. Kolom nilai guru format tuple `(LKPD,Kuis)` mis. `(60,96)`.

## 12. KEAMANAN
SECRET AES hanya di server & file statis build guru (jangan di repo publik lain). Guard server: kunci baris Username+Kode; duplikasi pasangan hanya bila RequestKolab 'Disetujui'; generate hanya status Aktif; login SHA-256 hibrid; peringatan password default di dashboard siswa.

## 13. PARAMETER FINAL LKPD INTERAKTIF + KUIS (ref: MTK-004 v2)
- Scaffold: setiap angka = input; kasus jenis segitiga wajib memuat baris urut sisi
  (a terpendek, b, c terpanjang) dan baris perbandingan bernomor: [c²] (?=,<,>) [a²+b²].
- Kesimpulan: 4 opsi = 1 benar + 3 distraktor MIRIP (beda simbol/operasi saja), urutan opsi di-shuffle per generasi.
- Skor LKPD: 1 poin/kotak; totalMax DINAMIS (jumlah check terdaftar); nilai=min(100, round(poin/totalMax*100));
  laporan menampilkan rincian per kegiatan. Jangan pernah hardcode totalMax.
- Peringatan isi-kosong: modal "Kegiatan n: x kotak kosong" + [Kembali Melengkapi]/[Tetap Evaluasi]; tidak memblokir keras.
- Kuis: urutan TIPE soal di-shuffle per sesi (disimpan di state); kesulitan mengikuti POSISI (pattern[k-1]).
  Q tipe B/S & menjodohkan tampil 10 item per layar (komposisi menjodohkan: 4 siku, 2 lancip, 2 tumpul, 2 bukan).
  Skor per soal kuis 20 (parsial per item/kotak). Payload kuis berlabel `#posisi(TIPE):skor`.
- Timer kuis: KESELURUHAN 15:00 (5×3 menit); minimal 60 detik per soal sebelum CEK/Lewati; auto-finish saat 00:00.
- Mesin pesan kuis: sebelum CEK = "Terisi x/y kotak — Lengkap/Belum lengkap";
  sesudah CEK = "Terisi x/y kotak — Tepat!/Belum tepat."; reset saat pindah soal (wajib, cegah pesan menempel).