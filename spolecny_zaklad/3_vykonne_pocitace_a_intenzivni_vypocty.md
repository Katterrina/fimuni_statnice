# 3. Výkonné počítače a intenzivní výpočty

> Superskalární, multijádrové a mnohojádrové (GPU, MIC) procesory, MIMD a SIMD paralelismus. Organizace paměti, sdílená a distribuovaná, cache koherence. Optimalizace kódu, optimalizující překladače. Distribuované systémy, topologie síťového propojení. Programování paralelních a distribuovaných systémů. (PA039)

## Úvod

Když se bavíme o superpočítačích, tak nás zajímá převážně v moderní době rychlost a možnost paralelizace řešení problémů.
Typicky již nemáme jen pouze jeden stroj, ale snažíme se ve skutečnosti vytvořit síť, která propujuje více výkonných strojů a snažíme se zoptimalizovat kód tak, aby běžel na strojích co nejrychleji v rámci celého superpočítače (clusteru).
V latence jednotlivých uložišť od registru přes cache, RAM paměti, paměti grafických karet, diskové uložiště až k nutnosti síťové komunikace se podílí na struktuře kódu, který superpočítač bude vykonávat.

## Procesory

### Kategorie

#### RISC (Reduced Instruction Set Computer)
- Jedná se o označení instrukční sady procesuru
- Procesor obsahuje pouze malou sadu základních instrukcí, které jsou provedeny rychle, typicky během jedné periody hodin (POZOR, to neznamená, že hodiny mají vysokou frekvenci oproti CISC) (POZOR^2 U moderních RISC není pravda, že 1 instrukce je za 1 periodu hodin)
- Pro programátora toto znamená:
  - Není tak náročné řešit překrývání mikroinstrukcí
  - Nižší spotřeba (implikující nižší nutnost aktivního chlazení, často dokonce žádná, např. mobil)
  - Nižší výkon
  - Vyšší efektivita (výkon/spotřeba) oproti CISC
- I když efektivita je vyšší, tak výkon nepřekračuje možnosti CISC. RISC tedy není typický pro superpočítače
- Příklady procesorů CISC: arm, RISC-V,...


#### CISC (Complex Instruction Set Computer)
- Jedná se o označení instrukční sady procesuru
- Procesor obsahuje pokročílé instrukce narozdíl od RISC, které typicky trvají i déle než 1 periodu hodin
- Pro programátora toto znamená:
  - Vyšší výkon
  - Vyšší spotřeba (implikující vyšší nutnost chlazení, typicky nutnost aktivního chlazení)
- CISC je používán u moderních superpočítačů
- Příkladem CISC instrukce může být:
  - násobení (Které je provedeno jako sada sčítacích operací)
  - logaritmus (Který je proveden sadou aproximací logaritmu)
- Příklady procesorů CISC: i386, x86_64, amd64, ...

#### Vector processors
> TODO: Dodělat

#### Streaming processors (e.g. GPU, TPU)
> TODO: Dodělat

#### Special processors
##### Programmable – FPGA (Field-programmable gate array)
Žádná velká bomba.
Jedná se o nevýkoné procesory, které se ale dají dobře masově vyrábět.
Typicky samotný čip je kompletní počítač, je programovatelný a má svoji volatilní paměť.
##### Static – ASICs (Application-specific integrated circuit)
> TODO:
