# Prolog programozás 6

## Különbség-listák

Listákat lehet két lista különbségeként ábrázolni, így pl. az `[1,2,3]` lista leírható úgy, mint az `[1,2,3,8]` és `[8]` listák különbsége, vagy az `[1,2,3]` és `[]` listáké, vagy általánosan az `[1,2,3|M]` és `M` különbségeként. A két tagot összekapcsolhatjuk pl. a `-` operátorral: `[1,2,3|M]-M`.

Ennek az ábrázolásnak nagy előnye, hogy közvetlenül tudunk hivatkozni a lista végére, ezért bizonyos műveleteket sokkal hatékonyabban meg lehet valósítani a segítségükkel, mint ha csak sima listákat használnánk.

Egy egyszerű példa erre a hozzáfűzés:

```prolog
hozzáfűz_kl(X-Y, Y-Z, X-Z).
```

Ehhez persze az kell, hogy argumentumként különbség-listákat kapjon:

```
?- hozzáfűz_kl([a,b,c|M]-M, [1,2]-[], L-[]).
M = [1, 2],
L = [a, b, c, 1, 2]
```

Mi is történik itt pontosan? Legjobban talán akkor látszik, ha a [3. leckében](prolog003.html) a listák bevezetésénél használt prefix jelöléssel írjuk fel:

```
?- hozzáfűz_kl(lista(a, lista(b, lista(c, M)))-M,
               lista(1, lista(2, vége))-vége,
               L-vége).
```

A szabály alapján az `M` és `lista(1, lista(2, vége))` egyesül, tehát a `hozzáfűz_kl` szabályban szereplő `X` értéke

```
lista(a, lista(b, lista(c, lista(1, lista(2, vége)))))
```

lesz, ami éppen a két lista összefűzése. Ugyanez többszörös összefűzésre is használható, pl.

```
?- hozzáfűz_kl([a,b|M1]-M1, [c,d|M2]-M2, L-M),
   hozzáfűz_kl(L-M, [e,f|M3]-M3, X-[]).
L = X, X = [a, b, c, d, e, f],
M = M2, M2 = [e, f],
M1 = [c, d, e, f],
M3 = []
```

A hozzáfűzésnek ezt a módját - mivel annyira egyszerű - nem szokás így külön szabályként felírni, hanem általában a programba közvetlenül építik be, ahogy azt mindjárt látni fogjuk a gyorsrendezésnél.

### Gyorsrendezés

Az előző leckében látott gyorsrendezés is sokkal hatékonyabbá tehető ezzel a technikával. Eredetileg így nézett ki:

```prolog
rendez2([], []).
rendez2([X|M], Y) :-
    szétoszt(X, M, Kicsi, Nagy),
    rendez2(Kicsi, K),
    rendez2(Nagy, N),
    hozzáfűz(K, [X|N], Y).
```

Írjuk át különbség-listák használatával!

```prolog
rendez2(X, Y) :- rendez2_kl(X, Y-[]).

rendez2_kl([], X-X).
rendez2_kl([X|M], Y-Z) :-
    szétoszt(X, M, Kicsi, Nagy),
    rendez2_kl(Kicsi, Y-[X|Y1]),
    rendez2_kl(Nagy, Y1-Z).
```

Látszik, hogy itt már nincsen szükség hozzáfűzésre, a két rekurzív hívás eleve "jó helyre" készíti a megoldásait. Ezért a varázslatért a változók egyesítése a felelős.

### Lista megfordítása

A `fordított` szabállyal már találkoztunk a 3. lecke 5. feladatában. Egy lehetséges megoldás a következő:

```prolog
fordított([], []).
fordított([X|M], Y) :- fordított(M, M1), hozzáfűz(M1, [X], Y).
```

Ez így nem túl hatékony. Ezt is megpróbálhatjuk vég-rekurziós formára hozni egy plusz (ún. *akkumulátor*) argumentum segítségével:

```prolog
fordított(X, Y) :- fordított(X, [], Y).

fordított([], Y, Y).
fordított([X|M], F, Y) :- fordított(M, [X|F], Y). 
```

Egy másik lehetőség, hogy különbség-listákat használunk:

```prolog
fordított(X, Y) :- fordított_kl(X, Y-[]).

fordított_kl([], X-X).
fordított_kl([X|M], Y-Z) :- fordított_kl(M, Y-[X|Z]).
```

Ha most összehasonlítjuk a két megoldást, látszik, hogy a kettő lényegében megegyezik, csak a vég-rekurziós változat harmadik és második paraméterét összevontuk egy különbség-listává.

### Feladatok

1. Írjátok át a 3. lecke 10. feladatában szereplő `lapít` szabályt hatékonyabbra különbség-listák használatával!

2. Oldjátok meg Dijkstra *holland zászló* problémáját: piros, fehér és kék színű elemek listáját rendezzétek át úgy, hogy a piros elemek után jöjjenek a fehérek, és végül a kékek, de ezen belül a sorrendjük ne változzon! Például:

       ?- holland([piros(alma),fehér(fal),kék(tenger),
                   piros(paprika),fehér(holló)], X).
       X = [piros(alma), piros(paprika), fehér(fal),
            fehér(holló), kék(tenger)]

   A megoldáshoz használjatok különbség-listákat!

## Kifejezések vizsgálata, struktúrák készítése és szétszedése

Egy kifejezés típusának (ld. 2. lecke) megvizsgálására a következő beépített szabályok adottak:

- `var(X)` : `X` változó (és nincs értéke)
- `nonvar(X)` : `X` nem változó vagy van értéke
- `atom(X)` : `X` atom
- `number(X)` : `X` szám
  - `integer(X)` : `X` egész szám
  - `float(X)` : `X` valós szám
- `atomic(X)` : `X` atom vagy szám
- `compound(X)` : `X` összetett struktúra

Egy struktúra ezen kívül szétszedhető az elemeinek listájára (fej + argumentumok), illetve visszaépíthető ezekből az `=..` segítségével:

```
?- f(a,b,c) =.. X.
X = [f, a, b, c]
?- X =.. [f, a, b, c].
X = f(a,b,c)
```

A funktort és aritást, illetve az egyes argumentumokat külön is le lehet kérdezni a `functor` ill. `arg` használatával:

```
?- functor(f(a,b,c), F, N).
F = f,
N = 3
?- arg(1, f(a,b,c), X).
X = a
?- arg(2, f(a,b,c), X).
X = b
```

Ezeknek a segítségével a struktúrákat tudjuk fix hosszú *tömbökként* kezelni, tehát olyan adattárolókként, amelyeknek tetszőleges eleme hatékonyan elérhető (ellentétben a listákkal, ahol az *n*-edik elem eléréséhez előbb végig kell mennünk az összes előtte levőn). Például a

```
?- functor(T, t, 10), arg(5, T, 42)
```

hatására a `T` egy olyan 10-elemű tömb lesz, aminek az 5. eleme a 42.

### Csere

Hogyan lehetne ezek segítségével megoldani a következő feladatot: egy kifejezésben egy másik (al)kifejezés minden előfordulását le akarjuk cserélni valami másra. Például:

```
?- lecserél(sin(x), t, 2*sin(x)*f(sin(x)), F).
F = 2*t*f(t)
```

Itt az "előfordulás" egyesíthetőséget jelent, tehát

```
?- lecserél(a+b, v, f(a,A+B), F).
A = a,
B = b,
F = f(a, v)
```

Három eset van:

1. A kifejezés és a lecserélendő alkifejezés megegyezik - az egészet cseréljük.
2. A kifejezés atom/szám - nincs mit csinálni.
3. A kifejezés egy struktúra; az argumentumaira kell elvégezni a cserét.

Ez alapján a három szabály:

```prolog
lecserél(X, Y, X, Y) :- !.
lecserél(_, _, Z, Z) :- atomic(Z), !.
lecserél(X, Y, Z, Z1) :-
    Z =.. [F|Arg],
    mindent_lecserél(X, Y, Arg, Arg1),
    Z1 =.. [F|Arg1].
```

A `mindent_lecserél` szabály egy lista minden elemére elvégzi a cserét:

```prolog
mindent_lecserél(_, _, [], []).
mindent_lecserél(X, Y, [Z|M], [Z1|M1]) :-
    lecserél(X, Y, Z, Z1),
    mindent_lecserél(X, Y, M, M1).
```

Ennek segítségével készíthetünk függvénykiértékelőt:

```
?- kiértékel(x*sin((x+y)/2), [x=1,y=2.14], X).
X = 0.9999996829318346
```

A második argumentumban `Szimbólum = szám` alakban vannak megadva a helyettesítési értékek.

```prolog
kiértékel(K, L, X) :- behelyettesít(K, L, K1), X is K1.

behelyettesít(K, [], K).
behelyettesít(K, [A=N|M], K2) :-
    lecserél(A, N, K, K1),
    behelyettesít(K1, M, K2).
```

### Feladatok

1. (*) Írjatok szabályt, ami egy összeadásokat tartalmazó kifejezést egyszerűsít úgy, hogy az ismeretleneket (ha lehet) összevonja és előre rakja, a többi összeadást pedig elvégzi!

       ?- egyszerűsít(1+1+a, E).
       E = a+2
       ?- egyszerűsít(1+a+4+2+b+c, E).
       E = a+b+c+7
       ?- egyszerűsít(3+x+x, E).
       E = 2*x+3

2. (*) Írjatok szabályt, ami eldönti, hogy egy kifejezés általánosabb-e egy másiknál! Az első argumentum akkor általánosítása a másodiknak, ha az első kifejezésben levő változóknak van olyan helyettesítési értéke, amivel pont a második kifejezést kapjuk meg.

       ?- általánosabb(X, c).           % X = c
       true
       ?- általánosabb(g(X), g(t(Y))).  % X = t(Y)
       true
       ?- általánosabb(g(t(Y)), g(X)).  % nincs Y, hogy t(Y) = X
       false
       ?- általánosabb(f(X,X), f(a,b)). % ellentmondás: X = a és X = b
       false

   (Feltehetjük, hogy a két kifejezés nem tartalmazza ugyanazt a változót.)

## Magasabb rendű szabályok

Vannak olyan szabályok, amelyek argumentumként egy másik szabályt (célt, kérdést) várnak. Ilyen volt például a tagadás, de van még néhány másik is.

### Listakezelés

Egy nagyon hasznos szabály a `maplist`, ami egy lista összes elemére megnézi, hogy teljesít-e egy adott szabályt, pl.

```
?- maplist(number, [3,4,5]).
true
?- maplist(number, [3,a,5]).
false
```

A szabály lehet többargumentumú is, ilyenkor több listát kell megadni, egyet minden argumentumhoz. A `mindent_lecserél` például megfogalmazható így:

```prolog
mindent_lecserél(X, Y, Z, Z1) :- maplist(lecserél(X, Y), Z, Z1).
```

Itt a `lecserél` első két argumentumát előre megadtuk, a maradék kettőt a `Z` és `Z1` listákból veszi ki.

### Összes megoldás

Gyakran előfordul, hogy az összes megoldásra kíváncsiak vagyunk. Ilyenkor ezeket egy listában le lehet kérni a `bagof` ("zsákja"), `setof` ("halmaza") vagy `findall` ("összeset keres") segítségével. Nézzük meg sorban őket!

A `bagof(X, P, L)` megkeresi az összes olyan `X`-et, amire `P` igaz, és `L` ezeknek a listája. Például:

```prolog
életkor(lica, 11).
életkor(mimi, 10).
életkor(dusa, 5).
életkor(zsófi, 5).
életkor(juli, 2).

?- bagof(X, életkor(X, 5), L).
L = [dusa, zsófi]
?- bagof(X, életkor(X, _), L).
L = [juli]
```

További megoldásokként megkapjuk életkorok szerint csoportosítva a többi gyereket is. Ha azt szeretnénk, hogy egyszerre az összes gyereket megkapjuk, akinek ismert az életkora (és nem csak azokat, akiknek azonos), akkor erre egy speciális jelölést kell használni:

```prolog
?- bagof(X, N^életkor(X, N), L).
L = [lica, mimi, dusa, zsófi, juli]
```

Az `N^` itt azt jelenti, hogy "van olyan `N`, amire igaz, hogy ...".

A `setof` nagyon hasonló ehhez, de az azonos elemekből csak egyet tart meg, például:

```prolog
?- bagof(N, X^életkor(X, N), L).
L = [11, 10, 5, 5, 2]
?- setof(N, X^életkor(X, N), L).
L = [2, 5, 10, 11]
```

Végül a `findall` olyan, mint a `bagof`, amikor a kifejezés egy változójának értéke sincs lekötve, tehát mintha minden (nem keresett) `V` változóhoz oda lenne írva a `V^`. Például:

```prolog
?- bagof(X, N^életkor(X, N), L).
L = [lica, mimi, dusa, zsófi, juli]
?- findall(X, életkor(X, _), L).
L = [lica, mimi, dusa, zsófi, juli]
```

Ezeknél a szabályoknál a cél lehet egy zárójelben levő összetett kifejezés is. Az első 10 háromszögszámot pl. így számíthatjuk ki:

```prolog
?- findall(Y, (között(1, 10, X), Y is X * (X + 1) // 2), L).
L = [1, 3, 6, 10, 15, 21, 28, 36, 45, 55].
```

### Feladat

Írjatok szabályt, ami a `bagof` segítségével megkeresi egy halmaz (lista) összes részhalmazát!

## Dinamikus szabályok

Szabályokat a program futása közben automatikusan is hozzá lehet adni az adatbázishoz, illetve ki lehet venni belőle. Ha a szabály már létezik, akkor ehhez az kell, hogy *dinamikusnak* legyen beállítva, pl.:

```prolog
:- dynamic foo/2. % 2 aritású
```

Ezután az alábbi módon lehet a `foo` szabályait módosítani:

- `asserta(foo([], _))` : az adatbázis elejére teszi a `foo([], _).` tényt.
- `assertz(foo(X, Y) :- X > Y)` : az adatbázis végére teszi a `foo(X, Y) :- X > Y.` szabályt.
- `retract(foo([], _))` : törli a `foo([], _).` tényt.
- `retractall(foo(_,_))` : törli az összes szabályt, aminek a feje egyesíthető a `foo(_,_)`-val.

Ezek segítségével például definiálhatjuk magunk is a `findall` szabályt:

```prolog
összes(X, Cél, L) :-
    Cél, assertz(tároló(X)), fail
    ; assertz(tároló(nincs_több)), összegyűjt(L).

összegyűjt(L) :-
    retract(tároló(X)), !,
    ( X = nincs_több, !, L = []
    ; L = [X|M], összegyűjt(M)
    ).

?- összes(X, életkor(X, _), L).
L = [lica, mimi, dusa, zsófi, juli]
```

Ez a megoldás feltételezi, hogy a `tároló` szabály még nem létezett, és hogy a keresett értékek közt nem szerepelhet a `nincs_több` atom. Ezt elkerülendő, ezeket a `$` prefix operátorral szokás megkülönböztetni, tehát `tároló(X)` helyett `$tároló(X)` és `nincs_több` helyett `$nincs_több` (vagy a zárójelet kiírva `$(tároló(X))` és `$(nincs_több)`).

Az önmagát módosító programok megértése nehéz, ezért az ilyen jellegű technikákat csak jól elkülönített programrészekben ajánlott alkalmazni.

## Vezérlés

A program folyásának vezérlésére már láttunk néhány módszert, mint a vágás (`!`) vagy a mindig igaz ill. hamis célok (`true`, `false` / `fail`). Itt van néhány további:

1. Ha egy szabály több megoldást is vissza tud adni, de mi csak az elsőt szeretnénk, a vágással le tudjuk tiltani a továbbiakat. Ez elég gyakori ahhoz, hogy van rá egy beépített szabály, a `once` ("egyszer"):

       once(P) :- P, !.

2. Amikor egy `P` változót célként használunk, mint a `once` vagy a tagadás definíciójában, akkor valójában a háttérben a `call(P)` ("hív") hívódik meg; ezt időnként ki is írják, hogy egyértelműbb legyen, mi történik. Amikor a `call`-nak több argumentuma van, ezeket a célhoz kapcsolandó további paraméterekként értelmezi, tehát pl.:

       ?- P = hozzáfűz([a,b]), call(P, [c,d], X), call(P, [x,y], Y).
       P = hozzáfűz([a, b]),
       X = [a, b, c, d],
       Y = [a, b, x, y].

3. A `(P -> Q; R)` jelentése: ha `P`, akkor `Q`, különben `R`. Tehát pl. az alábbi kettő ekvivalens:

       implikáció(X) :- foo(X) -> bar(X); baz(X).
   
   és

       implikáció(X) :- foo(X), !, bar(X).
       implikáció(X) :- baz(X).

   Egy kicsit érdekesebb, ha a `->` előtt is vannak célok, pl.:

       ?- között(1, 2, X), (X = 1 -> write(egy) ; write(kettő)), fail.
       egykettő
       false

   (A zárójelezésre szükség van, mert a `->` precedenciája nagyobb, mint a vessző operátoré.) Ugyanakkor

       ?- között(1, 2, X), (X = 1, !, write(egy) ; write(kettő)), fail.
       egy
       false
       
   A különbséget az okozza, hogy a vágás (`!`) az egész kérdésre vonatkozik, míg a `->` engedi, hogy a program visszamenjen a `között`-ig és újabb értékkel próbálkozzon.

4. A felhasználóval való kommunikációhoz gyakran van szükség végtelen ciklusra, ezt segíti a `repeat` ("ismétel"), amit így lehet definiálni:

       repeat.
       repeat :- repeat.

   Ez tehát mindig igaz, akárcsak a `true`, de ezt az igaz értéket végtelenszer generálja. Egy példa a használatára az alábbi program, ami a `read` segítségével kér be a felhasználótól számokat, és kiírja a négyzetüket, egészen amíg `stop`-ot nem kap:

       négyzetes :-
           repeat, read(X),
           ( X = stop, !
           ; Y is X * X, write(Y), fail
           ).

## Megjegyzések

Ez a dokumentum az alábbi könyv 6. és 8.5. fejezete alapján készült:

I. Bratko: *Prolog Programming for Artificial Intelligence*, 4th Ed., Pearson, 2011.

A különbség-listák tárgyalása részben az alábbi könyv 15.1. fejezete alapján készült:

L. Sterling, E. Shapiro: *The Art of Prolog*, 2nd Ed., MIT Press, 1994.

Edsger W. Dijkstra (1930-2002) nevét legtöbben a *Dijkstra-algoritmus* kapcsán ismerik. Ez egy úthálózatban (súlyozott gráfban) megkeresi két pont között a legrövidebb utat.
