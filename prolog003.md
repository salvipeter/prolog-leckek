# Prolog programozás 3

Ezúttal csak egy témánk van, de az nagyon fontos, és lesz hozzá sok feladat :)

## Listák

Ahogy a múltkor láttuk, egy Prolog kifejezés vagy egyszerű (konstans vagy változó), vagy egy fix aritású összetett struktúra. Gyakran előfordul azonban, hogy nem tudjuk előre, hogy hány adattal akarunk foglalkozni. Hogyan tudnánk dolgoknak egy listáját egy struktúrával kifejezni? Természetesen rekurzívan! Kezdjük először csak két dologgal:
```
lista(a, b)
```
Ha hármat akarunk beletenni egy listába, akkor megtehetnénk, hogy eggyel több argumentumot veszünk bele:
```
lista(a, b, c)
```
... de ez több okból sem igazán jó megoldás. Egyrészt ez így egy másik funktor lesz (hiszen más az aritása), másrészt ezzel még mindig csak *ismert* hosszúságú listákat tudnánk készíteni. Helyette csinálhatjuk viszont a következőt:
```
lista(a, lista(b, c))
```
Ezt tetszőlegesen lehet folytatni, pl.
```
lista(a, lista(b, lista(c, lista(d, e))))
```
Ez egyrészt azért jó, mert így mindig egy 2-aritású `lista` funktorunk van, másrészt ezzel tudunk olyat írni, hogy
```
lista(a, Maradék)
```
ami egy tetszőleges lista, aminek az első eleme `a`, vagy
```
lista(a, lista(b, Maradék))
```
ami egy olyan tetszőleges lista, aminek az első két eleme `a` és `b`.

Ennek így egy szépséghibája van: az utolsó elem kezelése kicsit más, mint a többié, és emiatt pl. az előző példákban a "tetszőleges" lista mégsem teljesen tetszőleges, mert nem lehet egy- ill. kételemű. Ezt azzal tudjuk megoldani, hogy bevezetünk egy olyan konstanst, amivel a lista végét jelöljük. Pl. az `a`, `b` és `c` elemeket tartalmazó lista ekkor így néz ki:
```
lista(a, lista(b, lista(c, vége)))
```

A szokásos Prolog jelölés a `lista` helyett a pont (`.`) funktor, a `vége` helyett pedig a szögletes zárójelpár (`[]`) atom. Az SWI-Prolog azonban a pontot lecserélte a kicsit furcsán kinéző `'[|]'` funktorra. Az előző listát tehát erre átírva ilyet kapunk:
```
'[|]'(a, '[|]'(b, '[|]'(c, [])))
```
Hát ez elég szörnyen néz ki. Szerencsére van rá egy egyszerűsített jelölés (ami kicsit magyarázza a funktor névválasztását):
```prolog
?- '[|]'(X, Y) = [X|Y].
true
```
Sőt, nem csak
```prolog
?- '[|]'(a, '[|]'(b, '[|]'(c, []))) = [a | [b | [c | []]]].
true
```
teljesül, hanem az egymásba ágyazást vesszővel lehet helyettesíteni:
```prolog
?- [a | [b | [c | []]]] = [a, b, c | []].
true
```
És végül van még egy utolsó "egyszerűsítés" (a szakszó erre a *syntactic sugar* avagy szintaktikus cukor): ha a függőleges vonal jobboldalán a `[]` van, akkor ezt el lehet hagyni:
```prolog
?- [a, b, c | []] = [a, b, c].
true
```
Ez tehát a Prolog *láncolt lista* struktúrája; a `[]` neve "üres lista", és egy `[X|Y]` listában az `X` a lista *eleje* (angolul *head*, fej), az `Y` pedig a lista *maradéka* (angolul *tail*, farok). A maradék mindig vagy egy lista, vagy az üres lista atom.

A gyakorlatban mindig ezt az egyszerű formát használjuk, de fontos érteni, hogy ez valójában egymásba ágyazott funktorokból áll, és ezért pl.
`[a, b, c] = [a | [b, c]] = [a, b | [c]] = [a, b, c | []]`.

## Műveletek listákon

### Tartalmazás

Az egyik legfontosabb kérdés, amit listákkal kapcsolatban feltehetünk, az az, hogy valami benne van-e:
```prolog
tartalmaz(X, [X|_]).
tartalmaz(X, [_|Maradék]) :- tartalmaz(X, Maradék).
```
Szavakban megfogalmazva, egy lista akkor tartalmaz valamit, (i) ha az az eleje, vagy (ii) ha a maradéka tartalmazza azt. Ezzel
```prolog
?- tartalmaz(b, [a, b, c]).
true
?- tartalmaz(b, [a, [b, c]]).
false
?- tartalmaz([b, c], [a, [b, c]]).
true
```
Emellett a
```prolog
?- tartalmaz(X, [a, b, c]).
```
kérdésre megkapjuk az `X = a`, `X = b` és `X = c` megoldásokat, sőt, meg is fordíthatjuk, és feltehetjük a kérdést, hogy "Milyen listák tartalmazzák `a`-t?"
```prolog
?- tartalmaz(a, L).
```
Erre a következő (jellegű) megoldásokat kapjuk:
```
L = [a | Maradék]
L = [X, a | Maradék]
L = [X1, X2, a | Maradék]
L = [X1, X2, X3, a | Maradék]
...
```
Az első egy tetszőleges lista, aminek az első eleme `a`; a második egy olyan, aminek a második eleme `a` és így tovább.

Feltehetünk összetett kérdéseket is, pl. milyen háromelemű listák vannak, amelyek tartalmazzák az `a`, `b` és `c` elemeket?
```prolog
?- L = [_, _, _], tartalmaz(a, L), tartalmaz(b, L), tartalmaz(c, L).
L = [a, b, c]
L = [a, c, b]
L = [b, a, c]
L = [c, a, b]
L = [b, c, a]
L = [c, b, a]
```

### Összefűzés

Két listát össze is tudunk csatolni. Legyen `hozzáfűz(L1, L2, L3)` igaz akkor, ha `L1` és `L2` egymás után rakva `L3`-at adja, pl.
```
?- hozzáfűz([a, b, c], [d, e], [a, b, c, d, e]).
true
```
Ha `L1` üres, akkor `L2` és `L3` megegyezik:
```prolog
hozzáfűz([], L2, L2).
```
Egyébként `L1` első eleme az `L3` első eleme lesz; az `L3` maradéka pedig az `L1` maradékának és az `L2`-nek az összefűzéséből adódik:
```prolog
hozzáfűz([X|M1], L2, [X|M3]) :- hozzáfűz(M1, L2, M3).
```

Annak ellenére, hogy onnan indultunk, hogy hogyan lehet két listát összefűzni, a kérdés ismét megfordítható, pl. "Hogyan lehet egy listát két listára osztani?"
```prolog
?- hozzáfűz(L1, L2, [a, b, c]).
L1 = [],
L2 = [a, b, c]
L1 = [a],
L2 = [b, c]
L1 = [a, b],
L2 = [c]
L1 = [a, b, c],
L2 = []
```

Vagy: "Igaz-e, hogy az `[a, b]` listával kezdődik az `[a, b, c]` lista?"
```prolog
?- hozzáfűz([a, b], _, [a, b, c]).
true
```

Megkereshetjük vele egy listában az előző és következő elemet:
```prolog
?- hozzáfűz(_, [Előző, már, Következő | _],
   [jan, feb, már, ápr, máj, jún, júl, aug, szep, okt, nov, dec]).
Előző = feb,
Következő = ápr
```

A `tartalmaz` szabályt is leírhatjuk a segítségével:
```prolog
tartalmaz(X, L) :- hozzáfűz(_, [X|_], L).
```
Szavakban: egy `L` lista akkor tartalmaz egy `X` elemet, ha szétválasztható két listára, amiből a másodiknak az első eleme `X`. (Az első lista ill. a második maradéka is lehet üres.)

### Hozzáadás és törlés

Új elemet egy lista elejéhez olyan könnyű hozzáadni, hogy erre nem is szokás külön szabályt írni, de ha akarunk, itt van:
```prolog
hozzáad(X, L, [X|L]).
```
A listák végére (hatékonyan!) beszúrni kicsit bonyolultabb, majd később lesz róla szó.

Hogyan kell törölni egy elem egy előfordulását egy listából?
```prolog
töröl(X, [X|M], M).
töröl(X, [Y|M], [Y|M1]) :- töröl(X, M, M1).
```
Tehát: ha `X` a lista első eleme, akkor a törlés után a lista maradékát kapjuk. Egyébként a lista első eleme és az eredmény első eleme megegyezik, és az eredmény maradéka pedig ugyanaz, mint az eredeti lista maradéka, amiből kitöröltük az `X`-et.

Egy példa a használatára:
```prolog
?- töröl(a, [a, b, a, a], L).
L = [b, a, a]
L = [a, b, a]
L = [a, b, a]
```
Itt a második és harmadik megoldás azonosnak tűnik, de valójában az egyik az eredeti listából az `a` elem második, a másik pedig a harmadik előfordulását törölte.

Mi történik, ha az "eredeti" listát vesszük ismeretlennek?
```prolog
?- töröl(a, L, [1, 2, 3]).
L = [a, 1, 2, 3]
L = [1, a, 2, 3]
L = [1, 2, a, 3]
L = [1, 2, 3, a]
```
Ahogy látszik, a "Mi az a lista, amiből ha kitörlünk egy `a`-t, akkor `[1, 2, 3]`-at kapunk?" kérdés egyszerűbben úgy fogalmazható meg, hogy "Mit kapunk, ha az `[1, 2, 3]` listába beleteszünk egy `a`-t?"

Ez van annyira hasznos, hogy adhatunk neki egy új nevet:
```prolog
betesz(X, L, L1) :- töröl(X, L1, L).
```

A törlés használható kiválasztásra is:
```prolog
?- töröl(X, [a, b, c], _).
X = a
X = b
X = c
```

Ha a törölni kívánt elem nem szerepel a listában, az eredmény `false` lesz. Ezt kihasználva ezzel is definiálhatjuk a `tartalmaz` szabályt:
```prolog
tartalmaz(X, L) :- töröl(X, L, _).
```

### Részlisták

Következőnek nézzük meg, hogy mikor része egy lista egy másiknak:
```prolog
?- részlista([c, d, e], [a, b, c, d, e, f]).
true
?- részlista([c, e], [a, b, c, d, e, f]).
false
```
Ahogy a második példából látszik, itt a "részét" nem úgy értelmezzük, hogy az első lista minden elemét tartalmazza a második (mint egy halmaznál), hanem hogy pontosan ugyanolyan sorrendben, más elemek közbeékelődése nélkül szerepelnek.

Ez a szabály könnyen megadható a `hozzáfűz` segítségével:
```prolog
részlista(R, L) :- hozzáfűz(_, L1, L), hozzáfűz(R, _, L1).
```
Tehát `R` akkor része `L`-nek, ha valamilyen listát elé- és utánafűzve megkapjuk `L`-et. A definíció ezt két részletben írja le: az első tag azt mondja, hogy az `L` az `L1` listára végződik; a második pedig azt, hogy ez az `L1` lista az `R`-el kezdődik. A vég kezdete pedig épp azt jelenti, hogy `R` valahol belül van `L`-ben (vagy valamelyik szélén, ha az itt alsóvonással jelölt listák egyike az üres lista).

Szokás szerint nézzük meg, mit kapunk a "fordított" felhasználásban:
```prolog
?- részlista(R, [a, b, c]).
R = []
R = [a]
R = [a, b]
R = [a, b, c]
R = []
R = [b]
R = [b, c]
R = []
R = [c]
R = []
```
Ahogy várható volt, ezzel megkapjuk az `[a, b, c]` lista összes részlistáját - az üreset többször is (miért? Tipp: nézzétek meg a `hozzáfűz(X, _, [a, b, c])` kimenetét).

### Permutációk

Két lista akkor *permutációja* vagy átrendezése egymásnak, ha ugyanazokat az elemeket tartalmazzák. Az üres listának csak az üres lista a permutációja:
```prolog
permutáció([], []).
```
Ha az első lista nem üres, akkor visszavezethetjük a feladatot az eggyel rövidebb listákra:
```prolog
permutáció([X|M], P) :- permutáció(M, L), betesz(X, L, P).
```
Tehát úgy kapjuk az első lista permutációját, hogy vesszük a maradék (`M`) egy permutációját (`L`), és ebbe betesszük az `X`-et.

Egy másik logika az lehet, hogy kiválasztunk (kitörlünk) egy elemet, a maradéknak vesszük egy permutációját, és az elejáre beszúrjuk a kitörölt elemet:
```prolog
permutáció(L, [X|P]) :- töröl(X, L, M), permutáció(M, P).
```

Ellenőrizzük, hogy működik-e! A
```prolog
?- permutáció([piros, zöld, kék], P).
```
kérdésre mindkettő visszaadja mind a 6 jó megoldást (bár különböző sorrendben) és közli, hogy nincs több. Viszont a fordított
```prolog
?- permutáció(P, [piros, zöld, kék]).
```
esetben az első verzió a 6 megoldás után végtelen rekurzióba kerül, a másodiknál pedig már az első után beragad. Meg lehet oldani, hogy mindig jó legyen, csak kicsit ki kell egészíteni, de mivel szimmetrikus, nincs rá igazán szükség.

### Feladatok

1. Írjatok egy szabályt, amivel ki lehet venni egy listából az utolsó 3 elemet!

       ?- kivesz3([a, b, c, d, e], [a, b]).
       true

2. Írjatok egy szabályt, amivel ki lehet venni egy listából az első és utolsó 3 elemet!

       ?- kivesz33([a, b, c, d, e, f, g], [d]).
       true

3. Írjatok egy szabályt, amivel megkaphatjuk egy lista utolsó elemét! Készítsetek két verziót, egyet `hozzáfűz`-zel, egyet anélkül!

       ?- utolsó([a, b, c], c).
       true

4. Írjátok meg a `páros_hosszú(L)` és `páratlan_hosszú(L)` szabályokat! Ezek akkor igazak, amikor az `L` lista páros- ill. páratlan számú elemből áll (nincs szükség számokra hozzá).
5. Írjatok egy szabályt, ami megállapítja, hogy két lista megfordítottja-e egymásnak!

       ?- fordított([a, b, c], X).
       X = [c, b, a]

6. Írjatok egy szabályt, ami megállapítja, hogy egy szó *palindróma*-e, azaz visszafele is ugyanaz-e!

       ?- palindróma([g,ö,r,ö,g]).
       true

7. Írjatok egy szabályt, amivel egy listát eggyel "elforgathatunk" úgy, hogy az első lista első eleme a második végére kerül!

       ?- forgat([a, b, c, d], [b, c, d, a]).
       true

8. Írjatok egy szabályt, ami a részhalmazságot vizsgálja! Le is lehessen vele generálni az összes részhalmazt! (A halmaz itt egy olyan rendezett lista, amiben minden elem egyszer fordul elő.)

       ?- részhalmaz([a, b, c], R). % az eredmény lehet más sorrendben
       R = [a, b, c]
       R = [a, b]
       R = [a, c]
       R = [a]
       R = [b, c]
       R = [b]
       R = [c]
       R = []

9. Írjatok egy szabályt, amivel ellenőrizhető, hogy két lista ugyanolyan hosszú!

       ?- ugyanolyan_hosszú([1, 2, 3], [a, b, c]).
       true

10. Írjatok egy szabályt, amivel ki lehet "lapítani" egy listát, tehát a belső listák elemeit kiviszi a legkülső szintre:

        ?- lapít([a, b, [c, d], [], [[[e]]], f], L).
        L = [a, b, c, d, e, f]

## Megjegyzések

Ez a dokumentum az alábbi könyv 3.1-3.2 fejezete alapján készült:

I. Bratko: *Prolog Programming for Artificial Intelligence*, 4th Ed., Pearson, 2011.

A Prolog beépítve tartalmaz néhány hasznos szabályt, amit most elkészítettünk:
- `tartalmaz` → `member`
- `hozzáfűz` → `append` (és `prefix(P, L) = append(P, _, L)`)
- `töröl` → `select`
- `permutáció` → `permutation` (de ez mindig jól működik)
- `utolsó` → `last`
- `fordított`  → `reverse`
- `ugyanolyan_hosszú` → `same_length`˛
- `lapít` → `flatten`
