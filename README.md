# Multi-Codec Image IP Core

Görüntü sıkıştırma (encode) ve açma (decode) işlemlerini yapan, FPGA üzerinde çalışacak
SystemVerilog/Verilog IP çekirdeği. Repo; RTL kodlarını, MATLAB referans modelini,
test vektörlerini, FPGA scriptlerini ve makale (LaTeX) kaynaklarını içerir.

---

## İçindekiler

1. [Proje Özeti](#proje-özeti)
2. [Dizin Yapısı ve Dosya Açıklamaları](#dizin-yapısı-ve-dosya-açıklamaları)
3. [Gereksinimler](#gereksinimler)
4. [Hızlı Başlangıç](#hızlı-başlangıç)
5. [Doğrulama Akışı](#doğrulama-akışı)
6. [Repo Kullanım Kuralları](#repo-kullanım-kuralları)
7. [Pull Request ile Değişiklik Yapma (ZORUNLU)](#pull-request-ile-değişiklik-yapma-zorunlu)
8. [Kod Stili ve İsimlendirme](#kod-stili-ve-isimlendirme)
9. [Lisans](#lisans)

---

## Proje Özeti

- **Amaç:** Birden fazla codec'i destekleyen, parametrik bir görüntü işleme IP çekirdeği.
- **Akış:** MATLAB golden model → test vektörleri → RTL simülasyonu → karşılaştırma → FPGA.
- **Çıktılar:** Sentezlenebilir RTL, doğrulama ortamı, FPGA prototipi, makale.

> TODO: Blok diyagramı ekle (`docs/figures/`), hedef codec listesini yaz.

---

## Dizin Yapısı ve Dosya Açıklamaları

```
multi-codec-ip/
├── README.md
├── LICENSE
├── .gitignore
├── .gitattributes
├── Makefile
├── docs/
├── rtl/
├── tb/
├── model/
├── vectors/
├── scripts/
├── constraints/
├── fpga/
└── paper/
```

### Kök dizin

| Dosya | Açıklama |
|---|---|
| `README.md` | Bu dosya. Proje tanımı, kurallar, kullanım. |
| `LICENSE` | Lisans metni (MIT / Apache-2.0 / CERN-OHL; danışman politikasına göre). |
| `.gitignore` | Git'in takip etmeyeceği dosyalar (Vivado proje klasörleri, simülasyon çıktıları, LaTeX ara dosyaları, MATLAB geçici dosyaları). |
| `.gitattributes` | Git LFS kuralları (büyük `.png`, `.bmp`, `.bit` dosyaları). |
| `Makefile` | Tek komutla işlemler: `make golden`, `make sim`, `make compare`, `make lint`, `make paper`. |

### `docs/` — Dokümantasyon

| Klasör | Açıklama |
|---|---|
| `docs/architecture/` | Blok diyagramları, arayüz tanımları, register map, veri yolu açıklamaları. |
| `docs/spec/` | Codec spesifikasyon notları, kaynak standart özetleri. |
| `docs/figures/` | Dokümanlarda ortak kullanılan şekiller (SVG/PDF). |

### `rtl/` — Donanım kodları (sentezlenebilir)

| Klasör / Dosya | Açıklama |
|---|---|
| `rtl/common/` | Ortak modüller: FIFO, line buffer, AXI-Stream wrapper vb. |
| `rtl/encoder/dct/` | Encoder tarafı dönüşüm (DCT) modülleri. |
| `rtl/encoder/quant/` | Kuantalama modülleri. |
| `rtl/encoder/entropy/` | Entropi kodlama (ör. Huffman/RLE) modülleri. |
| `rtl/decoder/entropy/` | Entropi çözme modülleri. |
| `rtl/decoder/dequant/` | Ters kuantalama modülleri. |
| `rtl/decoder/idct/` | Ters dönüşüm (IDCT) modülleri. |
| `rtl/top/` | Üst seviye modüller (`encoder_top.sv`, `decoder_top.sv`). |
| `rtl/filelist.f` | Derleme sırası listesi (simülatör, lint ve Vivado bu listeyi kullanır). **Yeni RTL dosyası eklenince buraya da eklenmelidir.** |

### `tb/` — Testbench'ler

| Klasör | Açıklama |
|---|---|
| `tb/unit/` | Modül bazlı testbench'ler (her RTL modülü için bir tane). |
| `tb/system/` | Uçtan uca encode → decode testleri. |
| `tb/common/` | Ortak BFM, task'lar, dosya okuma/yazma yardımcıları. |

### `model/` — Referans modeller

| Klasör / Dosya | Açıklama |
|---|---|
| `model/matlab/encoder/` | MATLAB encoder referans modeli. |
| `model/matlab/decoder/` | MATLAB decoder referans modeli. |
| `model/matlab/metrics/` | PSNR, SSIM, bitrate hesaplayan fonksiyonlar. |
| `model/matlab/gen_vectors.m` | RTL için test vektörü (hex) üretir. |
| `model/matlab/run_all.m` | Tüm modeli ve metrikleri çalıştırır. |

### `vectors/` — Test verisi

| Klasör | Açıklama |
|---|---|
| `vectors/input/` | Girdi test görüntüleri (küçük boyutlu; büyükler Git LFS ile). |
| `vectors/golden/` | MATLAB'in ürettiği referans çıktılar; RTL çıktısı bunlarla karşılaştırılır. |
| `vectors/README.md` | Vektörlerin formatı ve nasıl üretildiği. |

### `scripts/` — Otomasyon

| Dosya / Klasör | Açıklama |
|---|---|
| `scripts/vivado/` | `create_project.tcl`, `synth.tcl` gibi Vivado scriptleri. Proje bu scriptlerle yeniden üretilir. |
| `scripts/sim/` | xsim / Questa / Verilator çalıştırma scriptleri. |
| `scripts/compare.py` | RTL çıktısı ile golden çıktıyı karşılaştırır (bit-exact veya PSNR eşiği). |

### Diğer

| Klasör | Açıklama |
|---|---|
| `constraints/` | Ortak `.xdc` kısıt dosyaları. |
| `fpga/<board>/` | Karta özel top modül, constraint ve tcl dosyaları. |
| `paper/main.tex` | Makale ana dosyası; sadece `\input{sections/...}` içerir. |
| `paper/sections/` | Bölüm dosyaları (`intro.tex`, `method.tex`, `results.tex` ...). |
| `paper/figures/` | Makale şekilleri (tercihen PDF/SVG; MATLAB'dan `exportgraphics` ile). |
| `paper/tables/` | Tablo `.tex` dosyaları. |
| `paper/refs.bib` | Kaynakça. |
| `paper/Makefile` | `latexmk -pdf -outdir=build main.tex` ile derleme. |

---

## Gereksinimler

- Vivado (sürümü buraya yaz), MATLAB (sürümü buraya yaz)
- Verilator veya Icarus Verilog, Verible (lint)
- Python 3.x (`scripts/compare.py` için)
- LaTeX (TeX Live / MiKTeX) + `latexmk`
- Git ve Git LFS

---

## Hızlı Başlangıç

```bash
git clone <repo-url>
cd multi-codec-ip
git lfs install

make golden     # MATLAB golden vektörlerini üret
make lint       # RTL lint
make sim        # RTL simülasyonu
make compare    # RTL çıktısı vs golden
make paper      # Makaleyi derle
```

Vivado projesini oluşturmak için:

```bash
vivado -mode batch -source scripts/vivado/create_project.tcl
```

---

## Doğrulama Akışı

```
vectors/input/*.png
      │
      ▼
model/matlab/gen_vectors.m ──► vectors/golden/*.hex
                                      │
tb/system/tb_codec.sv (RTL sim) ──► out/*.hex
                                      │
                scripts/compare.py ◄──┘
```

---

## Repo Kullanım Kuralları

1. **`main` branch'ine doğrudan push YASAKTIR.** Her değişiklik (yeni dosya, dosya
   değişikliği, silme, klasör değişikliği, README/dokümantasyon dahil) **Pull Request**
   ile yapılır.
2. `main` her zaman derlenebilir ve simülasyonu geçer durumda olmalıdır.
3. Her PR **en az 1 kişi tarafından review edilip onaylanmalıdır**. Kendi PR'ını kendin merge etme.
4. PR'lar küçük ve tek konulu olmalı (ör. "DCT modülü ekle", "encoder testbench'i düzelt").
5. Merge öncesi reviewer'ın istediği düzeltmeler yapılmış olmalıdır.
6. **Commit etme:** Vivado proje klasörleri (`*.runs`, `*.cache`, `*.sim`, `.Xil`), simülasyon
   çıktıları (`*.vcd`, `*.wdb`), LaTeX ara dosyaları, MATLAB geçici dosyaları, büyük ikili dosyalar.
7. Bitstream (`.bit`) dosyaları repoya değil, GitHub **Releases** bölümüne yüklenir.
8. Yeni bir RTL dosyası eklenince `rtl/filelist.f` güncellenmelidir.
9. RTL'de yapılan her değişiklik için ilgili testbench çalıştırılmalı, gerekirse yeni test eklenmelidir.
10. Golden vektörler değişiyorsa PR açıklamasında nedeni belirtilmelidir.
11. Secret, lisanslı IP veya şifre içeren dosyalar commit edilmez.

### Branch isimlendirme

```
feature/<isim>-<konu>     örn: feature/kadir-dct-core
fix/<isim>-<konu>         örn: fix/reyis-quant-overflow
docs/<konu>               örn: docs/readme-update
paper/<konu>              örn: paper/results-section
```

### Commit mesajı formatı

```
<tür>: <kısa açıklama>

örnek:
feat: 2D DCT modülü eklendi
fix: quantizer taşma hatası giderildi
docs: README güncellendi
test: encoder system testbench eklendi
paper: results bölümüne PSNR tablosu eklendi
```

---

## Pull Request ile Değişiklik Yapma (ZORUNLU)

### 1) Güncel `main`'i al ve yeni branch aç

```bash
git checkout main
git pull origin main
git checkout -b feature/<isim>-<konu>
```

### 2) Değişikliklerini yap, commit'le

```bash
git status
git add <dosya-veya-klasör>      # tümü için: git add .
git commit -m "feat: kısa açıklama"
```

### 3) Branch'ini GitHub'a gönder

```bash
git push -u origin feature/<isim>-<konu>
```

> `main`'e push yapmaya çalışırsan branch protection hata verecektir. Bu beklenen davranıştır.

### 4) GitHub'da Pull Request aç

1. Repo sayfasına git. Push sonrası çıkan sarı **"Compare & pull request"** butonuna tıkla
   (çıkmazsa **Pull requests → New pull request** yolunu kullan).
2. `base: main` ← `compare: feature/<isim>-<konu>` olduğundan emin ol.
3. Başlık ve açıklamayı doldur (ne değişti, neden, nasıl test edildi).
4. Sağdan **Reviewers** kısmına ekip arkadaşını ekle.
5. **Create pull request** butonuna bas.

### 5) Review ve düzeltmeler

- Reviewer yorum bırakırsa aynı branch'te düzeltip tekrar commit + push yap; PR otomatik güncellenir.

### 6) Merge

- Onay sonrası **Squash and merge** (veya takımın seçtiği yöntem) ile merge et.
- Merge sonrası branch'i sil (GitHub'daki **Delete branch** butonu) ve lokalini güncelle:

```bash
git checkout main
git pull origin main
git branch -d feature/<isim>-<konu>
```

### PR şablonu

`.github/pull_request_template.md` dosyası oluşturup içine şunu koy; her PR'da otomatik gelir:

```markdown
## Ne değişti?
-

## Neden?
-

## Nasıl test edildi?
- [ ] `make lint`
- [ ] `make sim`
- [ ] `make compare`
- [ ] `make paper` (makale değiştiyse)

## Kontrol listesi
- [ ] `rtl/filelist.f` güncellendi (yeni RTL dosyası varsa)
- [ ] Gereksiz/büyük dosya commit edilmedi
- [ ] Dokümantasyon güncellendi (gerekirse)
```

### Repo sahibi için: `main` korumasını açma

Doğrudan push'u teknik olarak engellemek için (repo sahibi/admin yapar):

1. GitHub'da repo → **Settings → Branches → Add branch protection rule** (veya **Rules → Rulesets**).
2. **Branch name pattern:** `main`
3. Şunları işaretle:
   - **Require a pull request before merging**
   - **Require approvals** (en az 1)
   - **Do not allow bypassing the above settings** (admin'ler dahil herkes uysun)
4. **Create / Save changes**.

> Not: Özel (private) repolarda branch protection için GitHub Pro/Team gerekebilir. Ücretsiz
> planda bu mümkün değilse kuralı ekip olarak disiplinle uygulayın.

---

## Kod Stili ve İsimlendirme

- Dosya adı = modül adı (`dct_2d.sv` → `module dct_2d`).
- `snake_case` isimler, parametreler `BUYUK_HARF`.
- SystemVerilog için `.sv`, saf Verilog için `.v` uzantısı.
- Clock/reset: `clk`, `rst_n` (aktif-düşük, senkron/asenkron kararı `docs/architecture/` altında belirtilir).
- Her modülün başında kısa açıklama yorumu ve port tanımları olmalı.
- Lint uyarısız (veya bilinçli waiver'lı) kod beklenir.

---

## Lisans

Bkz. [LICENSE](LICENSE).
