# Rekonstrukcija i super-rezolucija slika primenom neuronskih mreža

## Članovi tima

* Staša Tonic 1115/2025
* Natalija Lazić 1080/2025

## Opis projekta

Cilj projekta je **rekonstrukcija slika visoke rezolucije na osnovu odgovarajućih slika niske rezolucije**, primenom različitih arhitektura neuronskih mreža za super-rezoluciju.

Problem je formulisan kao **Single Image Super-Resolution (SISR)** zadatak sa faktorom uvećanja **×4**, pri čemu model na osnovu slike niske rezolucije (**LR – Low Resolution**) generiše odgovarajuću sliku visoke rezolucije (**SR – Super Resolution**), koja se poredi sa originalnom slikom visoke rezolucije (**HR – High Resolution**).

U projektu su implementirana i analizirana četiri pristupa:

1. **SRCNN** – osnovni konvolucioni model za super-rezoluciju, korišćen kao baseline.
2. **SRResNet** – rezidualna neuronska mreža koja povećavanje rezolucije vrši unutar same arhitekture.
3. **ESRGAN** – generativna adversarijalna mreža zasnovana na RRDB blokovima, perceptualnom i adversarijalnom gubitku.
4. **ESRGAN sa pretraining fazom** – ESRGAN model kod kog se generator najpre trenira korišćenjem pixel-wise gubitka, a zatim dodatno trenira u adversarijalnom režimu.

Kao referentni metod koristi se **Bicubic interpolacija**.

Performanse modela evaluiraju se pomoću **PSNR (Peak Signal-to-Noise Ratio)** i **SSIM (Structural Similarity Index Measure)** metrika, uz dodatno vizuelno poređenje rekonstruisanih slika.

## Skup podataka

Za treniranje i validaciju modela korišćen je **DIV2K (DIVerse 2K Resolution)** skup podataka, namenjen zadacima super-rezolucije slika.

Korišćeni su:

* **DIV2K_train_HR** – 800 slika visoke rezolucije,
* **DIV2K_valid_HR** – 100 slika visoke rezolucije.

Originalne slike su različitih prostornih dimenzija, zbog čega se za treniranje modela iz njih izdvajaju manji patch-evi fiksne veličine.

Za finalnu evaluaciju koriste se standardni skupovi za super-rezoluciju:

* **Set5** – 5 test slika,
* **Set14** – 14 test slika.

Test skupovi nisu korišćeni tokom treniranja niti prilikom izbora najboljeg modela.

Zbog veličine, originalni skupovi podataka **nisu uključeni direktno u GitHub repozitorijum**.

Detaljno uputstvo za njihovo preuzimanje i očekivanu strukturu direktorijuma nalazi se u fajlu:

```text
data/README.md
```

Nakon preuzimanja repozitorijuma potrebno je preuzeti skupove podataka prema navedenom uputstvu i smestiti ih u odgovarajuće direktorijume unutar `data/` foldera.

Svi ostali podaci koji se koriste u projektu, uključujući LR–HR parove, trening i validacione skupove, kao i pripremljene test skupove, **generišu se tokom pokretanja odgovarajućih Jupyter Notebook sveski**.

## Analiza skupa podataka

U prvoj svesci izvršena je osnovna analiza **DIV2K** skupa podataka.

Analiza obuhvata:

* učitavanje i proveru strukture skupa podataka,
* analizu dimenzija originalnih slika,
* prikaz primera slika,
* proveru formata slika i broja kanala,
* proveru karakteristika skupa podataka.

Analizom je utvrđeno da slike nemaju jednake prostorne dimenzije, zbog čega je pre treniranja potrebno izvršiti pripremu podataka i formiranje LR–HR parova fiksnih dimenzija.

## Priprema podataka

Za potrebe treniranja iz originalnih DIV2K slika izdvajaju se **HR patch-evi dimenzija 128×128 piksela**.

Za svaki HR patch formira se odgovarajuća slika niske rezolucije primenom **Bicubic downsampling-a sa faktorom ×4**:

```text
HR patch: 128×128×3
        ↓ Bicubic downsampling ×4
LR patch: 32×32×3
```

Iz svake slike izdvojeno je po **10 patch-eva**, čime su formirani:

```text
Training set:   8000 LR–HR parova
Validation set: 1000 LR–HR parova
```

Dimenzije pripremljenih podataka su:

```text
X_train: (8000, 32, 32, 3)
y_train: (8000, 128, 128, 3)

X_val:   (1000, 32, 32, 3)
y_val:   (1000, 128, 128, 3)
```

LR slike predstavljaju ulaz modela, dok odgovarajuće HR slike predstavljaju ciljne vrednosti.

Za **SRCNN** model LR slike se pre prosleđivanja mreži dodatno povećavaju na dimenzije 128×128 korišćenjem Bicubic interpolacije.

**SRResNet** i **ESRGAN** primaju originalne LR slike dimenzija 32×32 i povećavanje rezolucije vrše unutar same neuronske mreže.

Pripremljeni trening, validacioni i test skupovi čuvaju se u `.npz` formatu kako bi mogli da se koriste u narednim sveskama.

Ovi fajlovi se **ne preuzimaju posebno**, već se generišu pokretanjem sveske za pripremu podataka.

## Podela podataka

Za treniranje, izbor najboljeg modela i finalnu evaluaciju koriste se međusobno odvojeni skupovi:

```text
DIV2K_train_HR
    ↓
Training set
8000 LR–HR parova

DIV2K_valid_HR
    ↓
Validation set
1000 LR–HR parova

Set5 + Set14
    ↓
Test skupovi
5 + 14 slika
```

**Trening skup** koristi se za optimizaciju parametara modela.

**Validacioni skup** koristi se za praćenje performansi tokom treniranja i izbor najbolje verzije modela.

**Set5** i **Set14** koriste se isključivo za finalnu evaluaciju istreniranih modela.

## Struktura projekta

```text
project/
│
├── data/
│   └── README.md
│
├── models/
│   ├── srcnn_best.pth
    ├── srresnet_best.pth
    ├── esrgan_standard_best.pth
    ├── esrgan_generator_pretrained.pth
    └── esrgan_best.pth
│
├── notebooks/
│   ├── 01_Data_Analysis.ipynb
│   ├── 02_Data_Preparation.ipynb
│   ├── 03_SRCNN_Baseline.ipynb
│   ├── 04_SRResNet.ipynb
│   ├── 05_ESRGAN.ipynb
│   ├── 06_ESRGAN_Pretrained.ipynb
│   └── 07_Model_Comparison.ipynb
│
├── requirements.txt
├── README.md
└── .gitignore
```

Folder `data/` sadrži `README.md` fajl sa instrukcijama i linkom za Drive za preuzimanje originalnih skupova podataka.

Podaci koji nastaju tokom pripreme skupa generišu se automatski pokretanjem odgovarajućih sveski.

Folder `models/` koristi se za čuvanje najboljih verzija istreniranih modela koje nastaju tokom treniranja.

## Preuzimanje projekta

Projekat je moguće preuzeti kloniranjem kompletnog GitHub repozitorijuma:

```bash
git clone https://github.com/NatalijaLazic/Rekonstrukcija-i-super-rezolucija-istorijskih-slika.git
```

Nakon toga potrebno je preći u direktorijum projekta:

```bash
cd Rekonstrukcija-i-super-rezolucija-istorijskih-slika
```

Repozitorijum sadrži sav kod, Jupyter Notebook sveske, konfiguracione fajlove i strukturu direktorijuma potrebnu za pokretanje projekta.

Originalni skupovi podataka nisu deo repozitorijuma zbog njihove veličine.

Njih je potrebno dodatno preuzeti prema instrukcijama iz:

```text
data/README.md
```

i smestiti u odgovarajuće direktorijume unutar `data/` foldera.

Svi ostali fajlovi sa podacima koji su potrebni u kasnijim fazama projekta generišu se tokom izvršavanja sveski.

## Podešavanje okruženja

Projekat je implementiran u **Python-u**, korišćenjem **Jupyter Notebook** sveski i **PyTorch** biblioteke.

> **Napomena:** Sveska `06_ESRGAN_Pretrained.ipynb` pokretana je u **Google Colab** okruženju uz **GPU akceleraciju**, dok su ostale sveske pokretane lokalno u **Visual Studio Code-u**.

Preporučeno je kreirati posebno virtuelno okruženje, na primer pomoću `conda`:

```bash
conda create -n super_resolution python=3.11
conda activate super_resolution
```

Sve biblioteke potrebne za pokretanje projekta navedene su u fajlu:

```text
requirements.txt
```

Nakon aktiviranja virtuelnog okruženja sve potrebne zavisnosti mogu se instalirati jednom komandom:

```bash
pip install -r requirements.txt
```

Na ovaj način instaliraju se biblioteke potrebne za:

* analizu i obradu podataka,
* rad sa slikama,
* treniranje neuronskih mreža,
* evaluaciju modela,
* vizuelizaciju rezultata,
* rad u Jupyter Notebook okruženju.

Ukoliko se za treniranje koristi **GPU**, potrebno je obezbediti odgovarajuću **CUDA podršku** kompatibilnu sa instaliranom verzijom **PyTorch-a**.

## Pokretanje projekta

Nakon preuzimanja repozitorijuma potrebno je:

1. kreirati i aktivirati Python okruženje,
2. instalirati potrebne biblioteke pomoću:

```bash
pip install -r requirements.txt
```

3. preuzeti originalne skupove podataka prema instrukcijama iz:

```text
data/README.md
```

4. preuzete skupove smestiti u odgovarajuće direktorijume unutar `data/` foldera,
5. pokrenuti Jupyter Notebook,
6. sveske izvršavati redosledom kojim su numerisane.

Svi dodatni podaci potrebni u kasnijim fazama projekta generišu se tokom izvršavanja prethodnih sveski.

Zbog toga, osim originalnih skupova navedenih u `data/README.md`, nije potrebno dodatno preuzimati prethodno pripremljene podatke.

## Redosled pokretanja sveski

Sveske su imenovane redosledom kojim ih treba pregledati i pokretati.

### 01_Data_Analysis.ipynb

Sveska za osnovnu analizu DIV2K skupa podataka.

Sadrži:

* učitavanje skupa podataka,
* proveru strukture direktorijuma,
* analizu dimenzija slika,
* prikaz primera slika,
* proveru formata slika i broja kanala,
* osnovnu analizu karakteristika skupa.

### 02_Data_Preparation.ipynb

Sveska za pripremu podataka za treniranje i evaluaciju modela.

Sadrži:

* izdvajanje HR patch-eva dimenzija 128×128,
* formiranje LR slika primenom Bicubic downsampling-a ×4,
* formiranje trening i validacionog skupa,
* normalizaciju podataka,
* vizuelnu proveru LR–HR parova,
* pripremu Set5 i Set14 test skupova,
* čuvanje pripremljenih podataka u `.npz` formatu.

Nakon pripreme dobijaju se LR slike dimenzija **32×32** i odgovarajuće HR slike dimenzija **128×128**.

Svi fajlovi generisani u ovoj svesci koriste se u narednim fazama projekta i nije ih potrebno posebno preuzimati.

### 03_SRCNN_Baseline.ipynb

Sveska u kojoj je implementiran **SRCNN (Super-Resolution Convolutional Neural Network)** kao osnovni model za poređenje.

Pre prosleđivanja modelu LR slike se povećavaju sa **32×32 na 128×128** pomoću Bicubic interpolacije.

SRCNN se sastoji od tri konvoluciona sloja:

```text
Conv2d(3 → 64, kernel=9)
        ↓
ReLU
        ↓
Conv2d(64 → 32, kernel=1)
        ↓
ReLU
        ↓
Conv2d(32 → 3, kernel=5)
```

Model uči da poboljša Bicubic rekonstrukciju i približi je originalnoj HR slici.

Za treniranje se koristi **MSE loss** i **Adam** optimizator.

Tokom treninga prate se trening i validacione greške, dok se najbolja verzija modela bira na osnovu performansi na validacionom skupu.

Finalna evaluacija vrši se na Set5 i Set14 skupovima, uz poređenje sa Bicubic interpolacijom.

Dobijeni rezultati su:

| Skup  | Metod   |    PSNR |   SSIM |
| ----- | ------- | ------: | -----: |
| Set5  | Bicubic | 26.7001 | 0.7899 |
| Set5  | SRCNN   | 27.9762 | 0.8244 |
| Set14 | Bicubic | 24.2562 | 0.6855 |
| Set14 | SRCNN   | 25.1358 | 0.7213 |

### 04_SRResNet.ipynb

Sveska u kojoj je implementiran **SRResNet** model.

Za razliku od SRCNN-a, SRResNet prima LR sliku u njenoj originalnoj rezoluciji i povećavanje rezolucije vrši **unutar same neuronske mreže**.

Arhitektura je zasnovana na:

* početnom konvolucionom sloju,
* rezidualnim blokovima,
* skip konekcijama,
* upsampling blokovima,
* završnom konvolucionom sloju za rekonstrukciju RGB slike.

Rezidualne veze omogućavaju efikasniji protok informacija i gradijenata kroz dublju mrežu.

Model je evaluiran na Set5 i Set14 skupovima:

| Skup  |    PSNR |   SSIM |
| ----- | ------: | -----: |
| Set5  | 28.8503 | 0.8468 |
| Set14 | 25.7176 | 0.7384 |

SRResNet sadrži ukupno **1.549.462 trenirajuća parametra**.

### 05_ESRGAN.ipynb

Sveska u kojoj je implementiran **ESRGAN (Enhanced Super-Resolution Generative Adversarial Network)**.

ESRGAN se sastoji od dve neuronske mreže:

```text
LR slika → Generator → SR slika

HR slika ─┐
          ├→ Discriminator
SR slika ─┘
```

Generator je zasnovan na **RRDB (Residual-in-Residual Dense Block)** arhitekturi.

U implementaciji se koristi **8 RRDB blokova**, pri čemu svaki RRDB sadrži tri RDB bloka.

Korišćeni parametri su:

```text
channels = 64
growth_channels = 32
residual scaling = 0.2
```

Povećavanje rezolucije ×4 realizuje se kroz dva uzastopna **×2 upsampling** koraka.

Diskriminator je implementiran kao VGG-stil konvoluciona mreža u kojoj se broj kanala postepeno povećava:

```text
64 → 128 → 256 → 512
```

Za treniranje generatora kombinuju se tri komponente funkcije gubitka:

* **pixel-wise loss**,
* **perceptual loss** zasnovan na VGG19 mreži,
* **relativistički adversarijalni loss**.

Perceptualni gubitak omogućava modelu da se ne fokusira isključivo na razliku između pojedinačnih piksela, već i na vizuelne karakteristike slike.

Najbolja verzija generatora bira se na osnovu **PSNR vrednosti na validacionom skupu**.

Rezultati na test skupovima su:

| Skup  | Metod   |    PSNR |   SSIM |
| ----- | ------- | ------: | -----: |
| Set5  | Bicubic | 26.7001 | 0.7899 |
| Set5  | ESRGAN  | 25.4406 | 0.7694 |
| Set14 | Bicubic | 24.2562 | 0.6855 |
| Set14 | ESRGAN  | 23.3115 | 0.6689 |

Pored numeričke evaluacije, izvršeno je i vizuelno poređenje rekonstrukcija kako bi se analizirala sposobnost ESRGAN-a da generiše izraženije teksture, ivice i sitne detalje.

### 06_ESRGAN_Pretrained.ipynb

Sveska predstavlja varijantu ESRGAN pristupa u kojoj je trening podeljen u **dve faze**.

#### Faza 1 – Pretraining generatora

Generator se najpre trenira nezavisno od diskriminatora, korišćenjem **L1 pixel-wise gubitka**.

Pretraining traje **15 epoha**, uz:

```text
Learning rate = 1e-4
Optimizer = Adam
Betas = (0.9, 0.999)
```

Cilj ove faze je da generator pre početka adversarijalnog treninga nauči stabilnu osnovnu rekonstrukciju slike.

Najbolja ostvarena validaciona vrednost u ovoj fazi iznosi:

```text
Best validation PSNR = 26.45 dB
```

#### Faza 2 – GAN fine-tuning

Nakon pretraining-a, najbolji generator iz prve faze koristi se kao početna tačka za adversarijalni trening sa diskriminatorom.

Tokom fine-tuning faze kombinuju se:

* pixel-wise gubitak,
* perceptualni gubitak,
* adversarijalni gubitak.

Najbolja ostvarena validaciona vrednost tokom GAN fine-tuning faze iznosi:

```text
Best validation PSNR = 24.77 dB
```

Rezultati finalnog ESRGAN generatora na test skupovima su:

| Skup  | Metod             |    PSNR |   SSIM |
| ----- | ----------------- | ------: | -----: |
| Set5  | Bicubic           | 26.7001 | 0.7899 |
| Set5  | ESRGAN Pretrained | 25.8836 | 0.7747 |
| Set14 | Bicubic           | 24.2562 | 0.6855 |
| Set14 | ESRGAN Pretrained | 23.8124 | 0.6896 |

Ovim pristupom ispituje se uticaj prethodnog pixel-wise treniranja generatora na stabilnost kasnijeg adversarijalnog treninga i kvalitet finalnih rekonstrukcija.

### 07_Model_Comparison.ipynb

## Rezultati

## Zaključak

## Literatura

1. Dong, C., Loy, C. C., He, K., & Tang, X. *Image Super-Resolution Using Deep Convolutional Networks*. IEEE Transactions on Pattern Analysis and Machine Intelligence, 2016.
   https://arxiv.org/abs/1501.00092

2. Ledig, C., Theis, L., Huszár, F., Caballero, J., Cunningham, A., Acosta, A., Aitken, A., Tejani, A., Totz, J., Wang, Z., & Shi, W. *Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network*. CVPR, 2017.
   https://arxiv.org/abs/1609.04802

3. Wang, X., Yu, K., Wu, S., Gu, J., Liu, Y., Dong, C., Loy, C. C., Qiao, Y., & Tang, X. *ESRGAN: Enhanced Super-Resolution Generative Adversarial Networks*. ECCV Workshops, 2018.
   https://arxiv.org/abs/1809.00219

4. Agustsson, E., & Timofte, R. *NTIRE 2017 Challenge on Single Image Super-Resolution: Dataset and Study*. CVPR Workshops, 2017.
   https://openaccess.thecvf.com/content_cvpr_2017_workshops/w12/html/Agustsson_NTIRE_2017_Challenge_CVPR_2017_paper.html

5. DIV2K Dataset – DIVerse 2K Resolution High Quality Images.
   https://data.vision.ee.ethz.ch/cvl/DIV2K/

6. Timofte, R., Agustsson, E., Van Gool, L., Yang, M.-H., Zhang, L., et al. *NTIRE 2017 Challenge on Single Image Super-Resolution: Methods and Results*. CVPR Workshops, 2017.
   https://openaccess.thecvf.com/content_cvpr_2017_workshops/w12/html/Timofte_NTIRE_2017_Challenge_CVPR_2017_paper.html

7. Wang, Z., Bovik, A. C., Sheikh, H. R., & Simoncelli, E. P. *Image Quality Assessment: From Error Visibility to Structural Similarity*. IEEE Transactions on Image Processing, 2004.
   https://ieeexplore.ieee.org/document/1284395

8. Jolicoeur-Martineau, A. *The Relativistic Discriminator: A Key Element Missing from Standard GAN*. ICLR, 2019.
   https://arxiv.org/abs/1807.00734

9. PyTorch Documentation.
   https://docs.pytorch.org/

10. Torchvision – VGG models documentation.
    https://docs.pytorch.org/vision/stable/models/vgg.html

11. scikit-image – Image Quality Metrics (PSNR and SSIM).
    https://scikit-image.org/docs/stable/api/skimage.metrics.html

12. Materijali sa vežbi kursa Mašinsko učenje, Univerzitet u Beogradu, Matematički fakultet.
    https://github.com/matf-ml/materijali-sa-vezbi-2026.git
