# 2. Statistika

> Důkladná znalost základních statististických metod (bodové odhady, intervaly spolehlivosti, testování statistických hypotéz). ANOVA. Neparametrické testy hypotéz. Mnohonásobná lineární regrese, autokorelace, multikolinearita. Analýza hlavních komponent (PCA). (MA012)

## Úvod

TODO

## Bodové odhady

- chceme odhadnout jeden číselný parametr distribuce, ze které pocházejí data
    - př. střední hodnota, rozptyl
- odhadujeme pomocí 

## t-test

## ANOVA (ANalysis Of VAriance)

- testujeme střední hodnoty $n$ skupin ($n \geq 3$, jinak t-test) odhaduté pomocí průměrů, tedy na hladině významnosti $\alpha$ testujeme hypotézy $H_0$ a $H_1$
    - $H_0$: všechny skupiny mají stejnou střední hodnotu
    - $H_1$: alespoň dvě skupiny mají rozdílné střední hodnoty 
- jednotlivé skupiny jsou stochasticky nezávislé
- $i$-tá skupina obsahuje $n_i$ měření $Y^i_1,...,Y^i_{n_i}$, náhodný vzorek z normální pravděpodobnostní distribuce
- proč ne opakovaný t-test? 
    - multiple testing problem
- předpoklady
    - homogenita rozptylu - rozptyly ve skupinách (alespoň přibližně) stejné
        testy: Levene’s test, Bartlett’s test
    - normalita - data ve skupinách normálně rozdělená
        - protože porovnáváme s kvantily F-rozdělení
        - testy: Lilliefors test, Shapiro–Wilk test (graficky: QQ-plot)
- pokud zamítneme $H_0$, zajímá nás, které dvojice skupin se liší
    - metody mnohonásobného porovnávání:
        - **Tukeyho metoda** - pokud máme podobné velikosti náhodných vzorků
        - **Scheffeho metoda** - hodně různorodé velokosti náhodných vzorků

### Model jednofaktorové ANOVA

- "jednofaktorová" - data na skupiny dělíme podle jednoho parametru
    - př. odrůda brambor, a ne ještě pole, na kterém byly pěstovány
- pozorování $Y_{ij}$ dopovídají modelu $M_A$ pokud $$Y_{ij} = \mu + \alpha_i + \epsilon_{ij} = \mu_i + \epsilon_{ij}$$ kde $\mu$ je celkový průměr, $\alpha_i$ je efekt skupiny $i$, $\mu_i$ je průměr skupiny $i$ a $\epsilon_{ij}$ jsou náhodné chyby
    - **TODO** střední hodnota vs průměr
    - ekvivalentní zápis hypotéz:
        - $H_0: \alpha_1 = ... = \alpha_n = 0$, $H_1: \exists i: \alpha_i \neq 0$
        - $H_0: \mu_1 = ... = \mu_n$, $H_1: \exists i,j: \mu_i \neq \mu_j$
    - za podmínky $H_0$ můžeme pozorování $Y_{ij}$ modelovat pomocí nulového modelu $M_0: Y_{ij}=\mu+\epsilon_{ij}$, který je submodelem $M_A$ 
- vícefaktorová ANOVA

## Mnohonásobná lineární regrese

## PCA

![Alt text](../obrazky/image.png)

- analýza hlavních komponent - statistická metoda pro redukci dimenzionality dat
- rotace ortonormální báze vektorového prostoru náhodných proměnných
    - hlavní komponenty (tedy nová báze) jsou nekorelované
        - vysvětlují co nejvíc z variability dat (rozptylu)