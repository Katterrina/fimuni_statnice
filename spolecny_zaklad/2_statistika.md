# 2. Statistika

> Důkladná znalost základních statististických metod ([bodové odhady](#bodové-odhady), [intervaly spolehlivosti](#intervaly-spolehlivosti-confidence-intervals-ci), [testování statistických hypotéz](#statistické-testy)). [ANOVA](#anova-analysis-of-variance). [Neparametrické testy hypotéz.](#neparametrické-testy) [Mnohonásobná lineární regrese](#mnohonásobná-lineární-regrese), [autokorelace](#autokorelace), [multikolinearita](#multikolinearita). [Analýza hlavních komponent (PCA).](#pca) (MA012)

## Úvod

### Proč statistika?

- deskriptivní (popisná) statistika
    - o daném jevu nic nevím, chci popsat získaná data
    - napříkla "jaký je náš průměrný věk" - každý mi řekne věk, spočítám průměr
- inferenční statistika
    - testování hypotéz, zobecňování zjištění na populaci
        - populace - celá množina, která nás zajímá (př. všichni lidi)
        - vzorek, reprezentativní vzorek - podmnožina, kterou testuji

### Hypotézy

- vždy, když chci ve statistice něco tvrdit, tak buď je to nějaký popis (průměr je X), nebo musím mít hypotézu
- hypotézy nelze potvrzovat, pouze zamítat
- **nulová hypotéza $H_0$** - nic se neděje, normální stav
    - to chci zamítnout
- **alternativní hypotéza $H_A$** - změna, to je to, co chci tvrdit
- $H_0$ a $H_A$ vždy komplementární, jedna z nich musí platit!

### P-hodnota

- pravděpodobnost, že pozorujete výsledky, které pozorujete (nebo extrémnější), za předpokladu, že nulová hypotéza platí
- p-hodnota pouze pomůže rozhodnout, jestli zamítáme nebo ne, žádným způsobem neměří "jak moc" je efekt významný
- nezamítnutí $H_0$ neznamená, že $H_0$ je pravdivá!
- pozor, p-hodnota není pravděpodobnost $H_0$, ani pravděpodobnost, že neplatí $H_A$


### Hladina významnosti a chyby I. a II. typu

![Chyby I. a II. typu](../obrazky/power.png)

- **hladina významnosti $\alpha$**
    - pravděpodobnost, že $H_0$ zamítáme, i když ve skutečnosti platí
- chyba I. typu, falešná pozitivita
    - pravděpodobnost, že $H_0$ zamítáme, i když ve skutečnosti platí
    - pravděpodobnost $\alpha$, můžeme si zvolit, vyšší s vyšším $\alpha$
- chyba II. typu, falešná negativita
    - pravděpodobnost, že $H_0$ nezamítáme, i když ve skutečnosti neplatí
    - pravděpodobnost jen odhadnutená, závisí na testu, počtu měření (čím víc tím menší), snižuje se s vyšším $\alpha$
- jak zvolit $\alpha$? 
    - příklad: testuji, zda má pacient nějakou nemoc a mám zahájit léčbu
        - $H_0$ zdravý pacient, $H_A$ nemocný pacient
        - možnost 1: léčba je zdarma a neinvazivní, stačí každé ráno vypít sklenici džusu, pokud ale nezačnu s léčbou včas, tak to může dopadnout špatně
            - nevadí, když občas řeknu i někomu, kdo nemoc nemá, že ji má, protože pití džusu nemůže uškodit
            - nevadí mi tolik chyby I. typu, chci se vyvyrovat chyb II. typu (nediagnostikuji někoho, kdo nemoc má)
            - volím vyšší hladinu významnosti
        - možnost 2: léčba je velmi nákladná, má ošklivé vedlejší účinky, nemoc je vážná a vyslechnutí diagnózy bude mít dramatický vliv na pacientovu psychickou pohodu, nemoc nepostupuje příliš rychle
            - chci se vyhnou chybě I. typu (neříkám zbytečně lidem, že tuhle nemoc mají)
            - volím nižší hladinu významnosti
        - v reálném světě složitější, než tento příklad    
    - typicky se používá 5% nebo 1% (v medicíně), ale je to určeno vlastně náhodně, konvence, není důvod, proč by 5% mělo být lepší než 4%

## Statistické testy

- matematický nástroj, který nám pomůže určitě, jestli na hladině významnosti $\alpha$ a při pozorování daných dat zamítáme $H_0$

### Postup

*Chceme získat znalost o nějaké vlastnosti světa, tedy pravděpodobnostní distribuce, ze které pochází data.*

1. rozmyslíme **hypotézy**, a model světa
2. vybereme nějakou vhodnou **testovou statistiku**
    - *číslo, které popisuje data*
    - důležité je, že víme, jaké rozdělení pravděpodobnosti bude tahle testová statistika mít, pokud platí $H_0$
        - př. průměr se bude pro hodně měření blížit střední hodnotě
3. zvolíme **hladinu významnosti** $\alpha$
4. spočítáme **hodnotu testové statistiky** pro naše data
    - tj. pokud je naše testová statistika výběrový průměr, spočítáme průměr
- **teď známe rozdělení pravděpodobnosti testové statistiky, pokud platí $H_0$ a její konkrétní hodnotu pro naše data**
5. spočítáme **p-hodnotu**, tedy pravděpodobnost, že pozoruji danou hodnotu testové statistiky za předpokladu, že platí $H_0$
    - můžu najít v tabulkách (kritická hodnota testové statistiky pro vybrané hladiny významnosti)
6. **zamítneme** $H_0$ ve prospěch $H_1$ pokud $p<\alpha$
 
### Předpoklady

- statistické testy mají předpoklady
- předtím, než se pustíme do testování nějakým testem, musíme jeho **předpoklady ověřit**
    - jinak si třeba budeme myslet, že víme, jaké rozdělení pravděpodobnosti má naše testová statistika, ale nebude to pravda
        - něco spočítáme (protože dosadit do vzorečků můžeme), ale to něco nemá žádnou hodnotu!

## Bodové odhady

- <https://bookdown.org/egarpor/inference/point.html>
- <https://portal.matematickabiologie.cz/index.php?pg=analyza-a-hodnoceni-biologickych-dat--statisticke-modelovani--zakladni-pojmy-matematicke-statistiky--bodove-odhady>
- chceme odhadnout jeden číselný parametr distribuce, ze které pocházejí data, na základě vzorku dat
    - př. střední hodnota, rozptyl
- odhadujeme pomocí výběrové charakteristiky (= funkce náhodného výběru)
- odhadovaný parametr $\theta$, odhad $\hat{\theta}$

### Vlastnosti bodových odhadů

- **nestrannost (nevychýlenost, nezkreslenost) - bias**
    - odhad je nestranný, pokud $\mathbb{E}(\hat{\theta})=\theta$
        - alternativně, pokud $Bias[\hat{\theta}]= \mathbb{E}(\hat{\theta})-\theta=0$
    - odhad je asymptoticky nestranný, pokud $\lim_{n \to \infty}\mathbb{E}(\hat{\theta})=\theta$
    - existuje mnoho dobrých odhadů, které nejsou nestranné
    - pro jeden parametr může existovat více nestranných odhadů
- **vydatnost (eficience)**
    - "aby bodové odhady byly rozloženy co nejtěsněji kolem odhadovaného parametru", tedy pokud budeme mít dva nestranné odhady, vybereme si ten s menším rozptylem
    - nestranný odhad, jehož rozptyl je nejmenší mezi všemi nestrannými odhady příslušného parametru, se nazývá nejlepší nestranný (eficientní) odhad
- **konzistence**
    - odhad je konzistentní (consistency in squared mean), pokud se s rostoucím rozsahem výběru ($n$) zpřesňuje, tedy:
        - $\hat{\theta}$ je asymptoticky nestranný, tedy $\lim_{n \to \infty}\mathbb{E}(\hat{\theta})=\theta$
        - $\lim_{n \to \infty}var(\hat{\theta})=0$
    - consistency in squared mean implies consistency in probability
    - polopatě: užívání konzistentních odhadů zaručuje
        - malou pravděpodobnost velké chyby v odhadu parametru, pokud rozsah výběru dostatečně roste
        - volbou dostatečně velkého počtu pozorování lze učinit chybu odhadu libovolně malou
- **dostatečnost**
    - odhad parametru je dostatečný, jestliže obsahuje veškerou informaci o sledovaném parametru, kterou může výbýrový soubor poskytnout (žádný jiný parametr neobsahuje větší množství informace o výběrovém souboru)
- bodový odhad je náhodná veličina
    - i v případě, že splňuje všechny vlastnosti výše, může se lišit od skutečného parametru $\to$ **výběrová chyba** $(\hat{\theta}-\theta)$
    - je-li $\hat{\theta}$ nezkresleným odhadem parametru $\theta$, můžeme měřit přesnost odhadu jeho směrodatnou odchylkou (nazývá se pak *střední chyba*)

### Příklady bodových odhadů

- střední hodnotu (expected value) $$\mathbb{E}[X]=\sum_x x\cdot P(X=x) = \int_{-\infty}^{\infty} x\cdot f(x) dx$$ odhadujeme pomocí **výběrového průměru** $$\overline{X} = \frac{1}{n}\sum_{i=1}^n X_i$$
    - nejlepší nestranný odhad střední hodnoty
    - dostatečný odhad střední hodnoty (využívá všechno z výběru, na rozdíl třeba od mediánu)
- rozptyl (variance) $$\begin{split} var(X) & = \mathbb{E}[(X-\mathbb{E}[X])^2] = \mathbb{E}[X^2]-(\mathbb{E}[X])^2 = \\ & = \sum_x (x-\mathbb{E}[X])^2\cdot P(X=x) = \\ & = \int_{-\infty}^{\infty} (x-\mathbb{E}[X])^2 \cdot f(x) \, dx \end{split}$$ odhadujeme pomocí **výběrového rozptylu** $$S_X^2= \frac{1}{n-1}\sum_{i=1}^n (X_i-\overline{X})^2$$
    - obecně není nestranný (unbiased) odhad
    - v případě náhodného výběru z normálního rozdělení je výběrový rozptyl nejlepším nestranným odhadem rozptylu
- směrodatnou odchylku (standard deviation) $$\sigma_X=\sqrt{var(X)}$$ odhadujeme pomocí **výběrové směrodatné odchylky** $$S_X=\sqrt{S_X^2}$$

### Intervaly spolehlivosti (confidence intervals, CI)

- intervalový odhad = arametr aproximujeme intervalem, v němž s velkou pravděpodobností daný parametr leží
- může být jednostranný nebo dvoustranný
- <https://bookdown.org/egarpor/inference/confint.html>
- říkají, "jak (ne)přesný" je bodový odhad nějakého parametru
    - "střední hodnota odhadu $\pm$ jeho rozptyl"
- hladina spolehlivosti: $1-\alpha$, kde $\alpha$ je hladina významnosti
    - v praxi hledáme kompromis mezi psolehlivostí a významností
- definice: $$CI_{1-\alpha}(\theta)=[T_{inferior}(x_1,\ldots,x_n),T_{superior}(x_1,\ldots,x_n)]$$ where $T_{inferior},T_{superior}$ jsou bodové odhady takové, že $$P(T_{inferior}\leq \theta \leq T_{superior})\geq 1-\alpha$$ pro všechny možné hodnoty $\theta$
- bootstraping nám může pomoct s konstrukcí různých druhů konfidenčních intervalů

#### Příklad: interval spolehlivosti pro střední hodnotu pro normální rozdělení

- předpokládejme sledovanou náhodnou veličinu $X$, známe její rozptyl $\sigma^2$
- víme, že pro dostatečně velký rozsah výbýru ($n\to \infty$) je rozdělení průměru
asymptoticky normální se střední hodnotou $\mu$ a rozptylem  $\sigma^2/n$, tedy
$$
\overline{X}\to N\left(\mu;\frac{\sigma^2}{n}\right)
$$
- definujeme náhodnou veličinu $Z$ jako $$ Z = \frac{\overline{X}-\mu}{\sqrt{\frac{\sigma^2}{n}}}=\sqrt{n}\cdot\frac{\overline{X}-\mu}{\sigma}$$ a víme, že $Z$ má normované normální rozdělení $Z\to N(0;1)$
- nechť $z_{a/2}$ a $z_{1-a/2}$ jsou $100\cdot (a/2) \%$ a $100\cdot (1-a/2) \%$ kvantily normovaného normálního rozdělení, pak můžeme tvrdit, že $$P(z_{a/2} < Z < z_{1-a/2})=1-\alpha$$ $$P(z_{a/2} < \sqrt{n}\cdot\frac{\overline{X}-\mu}{\sigma} < z_{1-a/2})=1-\alpha$$
- úpravou dostaneme oboustranný interval $$P(\overline{X}-\frac{\sigma}{\sqrt{n}}z_{a/2} < \mu < \overline{X}+\frac{\sigma}{\sqrt{n}}z_{1-a/2})=1-\alpha$$

## t-test

- porovnávání středních hodnot, buď jedné proti konstantě (jednovýběrový), nebo dvou proti sobě (dvouvýběrový)
- používá testovou statistiku $t$, která se spočítá z průměru (odhad střední hodnoty), výběrového rozptylu (odhad rozptylu) a počtu měření 
    - má známé t-rozdělení (viz obrázek)
        - jako jeho parametr se obvykle udává počet stupňů $\nu$ volnosti
        - pro $\nu \to \infty$ se blíží k normálnímu, pro $n>30$ už je to dobrá aproximace (takže pokud chceme použít tenhle test, tak chceme mít alespoň 30 měření)
    - nechť $X_i$ pro $i=1..n$ je $n$ naměřených hodnot náhodné veličiny $X$ (analogicky pro $Y$ a $m$)
    - jednovýběrový: $t=\frac{\bar{X}-\mu}{S_X/\sqrt{n}}$
        - $\bar{X} = \frac{1}{n}\sum_{i=1}^{n}X_i$ průměr, odhadujeme jím střední hodnotu
        - $S_X^2=\frac{1}{n-1}\sum_{i=1}^{n}(X_i-\bar{X})^2$ výběrový rozptyl, odhaduje rozptyl ($S_X$ se jmenuje výběrová směrodatná odchylka)
        - počet stupňů volnosti $n-1$
    - dvouvýběrový: $t =\frac{\bar{X}-\bar{Y}-\delta}{\sqrt{(n-1)S_X^2+(m-1)S_Y^2}}\sqrt{\frac{nm(n+m-2)}{n+m}}$ 
        - $\delta$ předpokládaný rozdíl středních hodnot, obvykle 0
        - počet stupňů volnosti $n+m-2$
- předpoklady: spojitá data, náhodný vzorek a nezávislá měření (pro závislá existuje párový test), homogenita rozptylu (pro párový t-test, rozptyly obou skupin jsou zhruba stejné), normalita

## ANOVA (ANalysis Of VAriance)

- testujeme střední hodnoty $a$ skupin ($a \geq 3$, jinak t-test) odhaduté pomocí průměrů $\to$ na hladině významnosti $\alpha$ testujeme hypotézy $H_0$ a $H_1$
    - $H_0$: všechny skupiny mají stejnou střední hodnotu
    - $H_1$: **alespoň dvě z $a$ skupin mají rozdílné střední hodnoty** 
- jednotlivé skupiny jsou stochasticky nezávislé
- $i$-tá skupina obsahuje $n_i$ měření $Y_{i1},...,Y_{in_i}$, náhodný vzorek z normální pravděpodobnostní distribuce
- proč ne opakovaný t-test? 
    - multiple testing problem - když opakujeme test hodněkrát, tak nám náhodou nakonec jednou vyjde, protože to je taky jenom pravědpodobnost
    - ANOVA garantuje $P(\text{type I error}) = \alpha$
- předpoklady
    - jednotlivá pozorování jsou nezávislá
    - homogenita rozptylu - rozptyly ve skupinách (alespoň přibližně) stejné
        - testy: Levene’s test, Bartlett’s test
    - normalita - data ve skupinách normálně rozdělená
        - protože porovnáváme s kvantily F-rozdělení
        - testy: Lilliefors test, Shapiro–Wilk test (graficky: QQ-plot)
- pokud zamítneme $H_0$, zajímá nás, které dvojice skupin se liší
    - metody mnohonásobného porovnávání:
        - **Tukeyho metoda** - pokud máme podobné velikosti náhodných vzorků
        - **Scheffeho metoda** - hodně různorodé velokosti náhodných vzorků

### Jednofaktorové ANOVA

- "jednofaktorová" - data na skupiny dělíme podle jednoho parametru
    - př. odrůda brambor (ale ne zárověň ještě pole, na kterém byly pěstovány)
- ekvivalentní zápis hypotéz:
    - $H_0: \alpha_1 = ... = \alpha_a = 0$, $H_1: \exists i: \alpha_i \neq 0$
    - $H_0: \mu_1 = ... = \mu_a$, $H_1: \exists i,j: \mu_i \neq \mu_j$
- pozorování $Y_{ij}$ dopovídají modelu $M_A$ pokud $$Y_{ij} = \mu + \alpha_i + \epsilon_{ij} = \mu_i + \epsilon_{ij}$$ 
    - $\mu$ celková střední hodnota, odhadujeme pomocí průměru všech měření $\hat{\mu}=\overline{Y}_{**}$
    - $\alpha_i$ efekt skupiny $i$, odhadujeme pomocí $\hat{\alpha_i}=\overline{Y}_{i*}-\overline{Y}_{**}$
    - $\mu_i = \mu + \alpha_i$ průměr skupiny $i$, odhadujeme pomocí $\hat{\mu_i}=\overline{Y}_{i*}$
    - $\epsilon_{ij}$ náhodné chyby s normálním rozdělením
    - za podmínky $H_0$ můžeme pozorování $Y_{ij}$ modelovat pomocí nulového modelu $M_0: Y_{ij}=\mu+\epsilon_{ij}$, který je submodelem $M_A$ 
        - v $M_0$ platí $\hat{\mu_i}=\hat{\mu}=\overline{Y}_{**}$
- součet čtverců vysvětluje variabilitu
    - celkový součet čtverců (total sum of squares) $\approx$ variabilita pozorování kolem celkového průměru $$S_T=\sum_{i=1}^a\sum_{j=1}^{n_i} (Y_{ij}-\overline{Y}_{**})^2 \sim \Chi^2(df_T=n-1)$$
    - regresní součet čtverců $\approx$  variabilita skupin kolem celkového průměru, tedy vysvětlení varability pomocí skupi $$S_A=\sum_{i=1}^a (\overline{Y}_{i*}-\overline{Y}_{**})^2 \sim \Chi^2(df_A=a-1)$$
    - reziduální součet čtverců $\approx$ variabilita každé skupiny okolo jejího průměru $$S_E=\sum_{i=1}^a\sum_{j=1}^{n_i} (Y_{ij}-\overline{Y}_{i*})^2 \sim \Chi^2(df_E=n-a)$$
    - platí $S_T=S_A+S_E$
    - součty čtverců využijeme k porovnání $M_0$ a $M_A$, spočítáme testovou statistiku s Fisher-Snedecor probability distribution

#### ANOVA pro porovnávání modelů

- porovnávání vnořených lineárních modelů (porovnání model-submodel)
- hypotézy
    - $H_0$: $M_1$ může být zjednodušen na $M_2$ (modely jsou ekvivalentní a $M_2$ má méně parametrů)
    - $H_A$: $M_2$ není validní


### Vícefaktorová ANOVA

- model pro dvoufaktorovou ANOVU: $$Y_{ijk}=\mu_{ij}+\epsilon_{ijk}$$ kde $\epsilon_{ijk}$ jsou stochasticky nezávislé náhodné proměnné z normální pravděpodobnostní distribuce s nulovou střední hodnotou
- máme více modelů, pro dvoufaktorovou anovu uvažujeme vliv
    - faktoru A, B a jejich interakce AB
    - faktoru A, faktoru B
    - faktoru A
    - faktoru B
    - nulový model, faktory nemají vliv
- porovnáváme všechny možné modely pro různé kombinace vlivů (jen pro A a B zvlášť nejsou porovnatelné, protože to není submodel)
- $S_T=S_{AB}+S_A+S_B+S_E$

## Neparametrické testy

### Parametrické metody

- většina statistických testů jsou paramterické metody, tedy používají např. normální distribuci pro modelování náhodných dat
- na základě pozorovaných dat spočítáme bodové odhady nějaký parametrů, konfidenční intervaly, provádíme statistické testy
    - všechny výpočty jsou založené na tom, že pozorovaná data pochází z nějaké parametrické rodiny
    - typické předpoklady: interval nebo ratio data type, normalita vzorku, homogeita rozptylu, ...
- pravděpodobnostní model popisuje možné výsledky náhodného procesu
    - model je zásadní pro určení (ne)jistoty bodového nebo intervalového odhadu nebo určení kritické oblasty a výpočet p-hodnoty

### Neparametrické metody

- použijeme v případě, že předpoklady parametrických metod (normalita dat, homogenita rozptylu, interval/ratio data) nejsou splněny
- příklady
    - neparametrické regresní modely
    - neparametrické testy
    - neparametrické odhady hustoty pravděpodobnosti
- uspořádaný náhodný výběr je náhodný vektor $(X_{(1)},X_{(2)} \ldots X_{(n)})$ takový, že $X_{(1)}\leq X_{(2)}\leq  \ldots \leq X_{(n)}$, náhodná proměnná $X_{(i)}$ se nazývá $i$-tá pořádková statistika
- pořadí (rank) $R_i$ náhodné proměnné $X_i$ odpovídá pořadí $X_i$ v uspořádaném náhodném vzorku
    - využijeme pro pořadové testy

### Znaménkový test

- pořadový test
- nechť $(X_1,\ldots,X2)$ je náhodný výběr ze spojité pravděpodobnostní distribude s mediánem $\tilde{x}$
    - víme $\forall i: P(X_i<\tilde{x})=P(X_i>\tilde{x})=1/2$
- **otázka: je medián roven $x_0\in \mathbb{R}$?**
    - $H_0: \tilde{x}=x_0$
    - $H_A: \tilde{x}\neq x_0$
- test:$$T^+=|\{i:X_i>x_0\}|=\sum_i Z_i$$ kde $Z_i$ je indikátorová náhodná proměnná (nabývá hodnoty 1 pokud $X_i>x_0$)
    - $H_0$ zamítáme, pokud $T^+\leq k_\alpha$ nebo $T^+\geq n-k_\alpha$ kde $k_\alpha$ je největší číslo z množiny $\{0,\ldots, n\}$ takové, že $$P(T^+\leq k_{\alpha})=\frac{1}{2^n}\sum_{i=0}^{k_\alpha}\binom{n}{i}\leq\alpha/2$$ a $$P(T^+\geq n-k_{\alpha})=\frac{1}{2^n}\sum_{i=n-k_{\alpha}}^{n}\binom{n}{i}\leq\alpha/2$$
    - asymptotická varianta ($n\geq 20$): platí-li $H_0$, pak testová statistika $U=\frac{T^+-\mathbb{E}[T^+]}{\sqrt{var(T^+)}}$ má asymptoticky standardní normální rozdělení
- **párový znaménkový test**: odečteme od sebe páry, pak nepárový

### Jednovýběrový Wilcoxonů test (signed-rank Wilcoxon test)

- předpokládáme ordinální data
- nechť $(X_1,\ldots,X2)$ je náhodný výběr ze spojité pravděpodobnostní distribude se symetrickou hustotou pravděpodobnosti $f(x)$, medián $\tilde{x}$
    - víme $\forall i: P(X_i<\tilde{x})=\int_{-\infty}^\tilde{x}f(x)dx=P(X_i>\tilde{x})=\int_{\tilde{x}}^{\infty}f(x)dx=1/2$
- **otázka: je medián roven $x_0\in \mathbb{R}$?**
    - $H_0: \tilde{x}=x_0$
    - $H_A: \tilde{x}\neq x_0$
- test:
    1. nechť $Y_i=X_i-x_0$
    2. seřadíme  $Y_i$: ${|Y_i|}_{(1)}\leq \ldots \leq {|Y_j|}_{(n)}$, nechť $R_i^+# je pořadí $Y_i$ v seřazené sekvenci
    3. spočítáme sum of ranks pro kladná a záporná $Y_i$ $$T^+=\sum_{Y_i>0}R_i^+, T^-=\sum_{Y_i<>>0}R_i^+$$
    4. zaímáme na základě $min(T^+,T^-)$ porovnané s kritickou hodnotou
- asymptoticky stejně jako výše znaménkový test
- silnější než znaménkový test

#### Dvouvýběrový

- testujeme rovnost distribučních funkcí
- podobně, jako výše, seřadíme všechno dohromady a pak uděláme součty ranků

### Kruskal-Wallis test

- "neparametrická ANOVA"
- použijeme, pokud jsou porušeny podmínky ANOVY, ale náhodné výběry jsou nezávislé
- hypotézy
    - $H_0: F_1(x)=F_2(x)=\ldots=F_n(x)$
    - $H_A: \exists i,j: i\neq j, F_i(x)\neq F_j(x)$
- test:
    - spojíme pozorování do jednoho vektoru $(Y_{11},\ldots,Y_{an_a})$
    - seřadíme $Y_{(1)}\leq Y_{(2)}\leq  \ldots \leq Y_{(n)}$, $n=\sum_{i=1}^a n_i$
    - nechť $R_{ij}$ je pořadí $Y_{ij}$
    - sečteme pořadí v kategorii, z nich získáme testovou statistiku a porovnáme s kritickou hodnotou

## Mnohonásobná lineární regrese

- nechť máme náhodnou proměnnou $Y$ závislou na náhodných proměnných $X_1,\ldots,X_l$ takovou, že $$Y=\beta_0+\sum_{i=1}^l \beta_i X_i +\epsilon= X\cdot\beta+\epsilon$$ kde
    - $Y$ vektor, v každém řádku jedno pozorování
    - $X$ matice plánu, první sloupec 1, další sloupce náhodné proměnné $X_i$ (všechny v první mocnině); matice má plnou hodnost, tj. sloupce jsou lineárně nezávislé
    - $\beta$ regresní koeficienty
    - $\epsilon$ nesystematické nekorelované náhodné chyby s homogenním rozptylem
- jak najít nejlepší lineární aproximaci $\hat{Y}$ (aproximaci s nejmenším rozptylem), když máme $n$ pozorování? 
    - hledáme taková $\beta$, že minimalizují $$S(\beta)=\sum_{i=1}^n[Y_i-\sum_{j=0}^l \beta_j X_{ij}-\beta_0]^2=(Y-X\beta)'(Y-X\beta)$$
    - odhad metodou nejmenších čtverců $\hat{\beta}_{OLS}=(X'X)^{-1}(X'Y)$
    - predikované hodnoty $\hat{Y}=X\hat{\beta}_{OLS}$
    - rezidua $ri=Y_i-\hat{Y}_i$, reziduální součet čtverců pak je $$S_E=S_{\hat{\beta}_{OLS}}=\sum_{i=1}^n r_i^2$$

## Autokorelace

- obvykle v modelech předpokládáme nekorelované chyby
- autokorelace znamená, že chyby jsou korelované, tedy $\rho(\epsilon_i,\epsilon_j)\neq 0$
- kovarianční matice náhodných chyb není diagonální s homogenními rozptyly
    - formálně: $var(\epsilon)\neq \sigma^2 I_n$
- pokud použijeme OLS (ordinary least squares), může vést k mylnému dojmu, že některé koeficienty jsou signifikantní $\to$ spočítat to jde, ale je to blbě
- jak zjistit autokorelaci: Durbin-Wasonův test, testová statistika $D$
- řešení: extended linear regression model $Y=X\beta+\epsilon,\mathbb{E}[\epsilon]=0, var(\epsilon)=\sigma^2V$, kde $var(\epsilon)$ je symetrická a pozitivně definitní matice
    - zobecněná metoda nejmenších čtverců (Atkienův odhad) $$\hat{\beta}_{GLS}=(X'V^{-1}X)^{-1}X'V^{-1}Y$$
    - $var(\hat{\beta}_{GLS})=\sigma^2(X'V^{-1}X)^{-1}$
    - $V$ kovarianční matice je symetrická a pozitivně definitní
- autoregrese - speciální vztah mezi následujícími náhodnými chybami

## Multikolinearita

- multikolinearita je vzájemná lineární závislost prediktorů (vysvětlujících proměnných) v regresním modelu, tj. některé sloupce matice plánu jsou lineárně závislé
    - v praxi: determinant $X'X$ je blízký nule, matice je skoro singulární $\to$ neexistuje inverze $\to$ nelze počítat $\hat{\beta}_{OLS}= (X'X)^{-1}X'Y$ (signulární nelze, skorosingulární způsobuje numerickou nestabilitu)
        - důsledky: obtížné vyjádření vlivu jednotlivých vysvětlujících proměnných, koeficienty vyjdou nevýznamné, protože vysoké hodnoty rozptylů způsobí široké intervaly spolehlivosti
- důvod: nadbytečné vysvětlující proměnné, například kvůli nepstávnému výběru prediktorů
    - může být způsobeno vztahem, který neznáme, například dvě vysvětlující proměnné závisí na třetí, kterou neuvažujeme
    - například v časových řadách je podobný vývoj dvou veličin dostatečným důvodem pro vznik multikolinearity
- řešení: vybereme "správné" prediktory
    - zkoušíme různé modely a porovnáváme je na základě daného kritéria, př. AIC, BIC (ne reziduální součet čtverců nebo koeficient determinace, v obou bychom stále jenom přidávali parametry)
    - metoda postupné regrese
        - dopředná - začneme s prázným modelem, v každém cyklu testujeme modely s +1 prediktory
        - zpětná - začneme s maximálním modelem, v každém cyklu odebereme nejméně významný prediktor
        - obousranná - v každém kroku zkusíme přidat i odebrat prediktor, vybereme nejlepší variantu


## PCA

- analýza hlavních komponent - statistická metoda pro redukci dimenzionality dat
- rotace ortonormální báze vektorového prostoru náhodných proměnných
    - hlavní komponenty (tedy nová báze) jsou nekorelované
    - báze je optimální - 1. PC vysvětluje co nejvíc z variability dat (rozptylu), 2. PC vysvětluje co nejvíce variability nevysvětlené 1. PC $\ldots$ vysvětlený rozptyl postupně klesá

![PCA](../obrazky/image.png)


- nechť $X$ je matice $n$ pozorování $k$ centrovaných náhodných proměnných kde $n>k$, $i$-tý řádek reprezentuje bod v $\mathbb{R}^k$, $X_{i1}, \ldots, X_{ik}$ jsou jeho souřadnice
$$X=
\left(\begin{array}{ccc} 
X_{11} & \ldots & X_{1k} \\
\ldots & \ldots & \ldots \\
X_{n1} & \ldots & X_{nk} \\
\end{array}\right)
$$

### Aproximace $x^*$

- chceme zkonstruovat projekci vektoru $x$ z prostoru $\mathbb{R}^k$ do podprostoru $\mathbb{R}^r$ $\to$ aproximaci $x^*$
- zvolíme ortonormální bázi $U = (u_1,\ldots,u_k) \in \mathbb{R}^k$ vektorového prostoru $\mathbb{R}^k$
    - $U$ je ortogonální matice, tedy $U'U=I_k$ a $U'=U^{-1}$
    - každý bod ve vektorovém prostoru $\mathbb{R}^k$$ můžeme vyjádřit jako $x= a_1u_1+\ldots+a_ku_k=Ua$ kde $a_{1\ldots k}$ jsou souřadnice převádějící $x$ ze standardních souřadnic $a=U'x$

