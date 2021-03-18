# Prolog programozás 5

## Rendezések

Egy klasszikus - és nagyon hasznos - programozási feladat számok listájának növekvő sorrendbe rendezése. Erre rengeteg módszer van, itt csak a legfontosabbak közül nézünk meg néhányat.

### Beszúrásos rendezés (*insertion sort*)

Az első algoritmus a *beszúrásos rendezés*. Az ötlet a következő: ha van egy listám, akkor azt szétszedem az első elemre és a maradékára, az utóbbit (rekurzívan) rendezem, és végül az első elemet beszúrom a "megfelelő" helyre.

```prolog
rendez1([], []).
rendez1([X|M], Y) :- rendez1(M, M1), beszúr(X, M1, Y).
```

A megfelelő helyre való beszúrásnál pedig egyszerűen addig megyünk, amíg egy legalább akkora elemet nem találunk:

```prolog
beszúr(X, [], [X]).
beszúr(X, [Y|M], [X,Y|M]) :- X =< Y.
beszúr(X, [Y|M], [Y|M1]) :- X > Y, beszúr(X, M, M1).
```

Próbáljuk ki!

```
?- rendez1([4,1,9,2,6,0,3,2,5,1], X).
X = [0, 1, 1, 2, 2, 3, 4, 5, 6, 9]
```

Ennek az algoritmusnak egy nagy előnye, hogy akkor is jól használható, ha az adatokat folyamatosan kapjuk, hiszen ha már van egy rendezett listánk, akkor utána mindig elég a `beszúr` műveletet használni. Egy másik jó tulajdonsága, hogy egy már rendezett sorozatnál nem végez plusz munkát, épp csak ellenőrzi, hogy a lista tényleg jó sorrendben van:

```
?- trace, rendez1([1,2,3], X).
Call:rendez1([1, 2, 3], X)
 Call:rendez1([2, 3], X1)
  Call:rendez1([3], X2)
   Call:rendez1([], X3)
   Exit:rendez1([], [])
   Call:beszúr(3, [], X2)
   Exit:beszúr(3, [], [3])
  Exit:rendez1([3], [3])
  Call:beszúr(2, [3], X1)
   Call:2=<3
   Exit:2=<3
  Exit:beszúr(2, [3], [2, 3])
 Exit:rendez1([2, 3], [2, 3])
 Call:beszúr(1, [2, 3], X)
  Call:1=<2
  Exit:1=<2
 Exit:beszúr(1, [2, 3], [1, 2, 3])
Exit:rendez1([1, 2, 3], [1, 2, 3])
X = [1, 2, 3]
```

Ugyanakkor ha nincs szerencsénk, nem túl jó hatásfokú: ha a lista *n* hosszú, akkor kb. *n^2* összehasonlítást végez a legrosszabb esetben.

### Gyorsrendezés (*quicksort*)

A gyorsrendezésnél kiválasztunk egy tetszőleges elemet (pl. az elsőt), és a többit két részre osztjuk aszerint, hogy nagyobb-e ennél vagy sem. Ezután a két részt külön-külön rendezzük (rekurzívan), majd az eredmény ezek összefűzéséből adódik:

```prolog
rendez2([], []).
rendez2([X|M], Y) :-
    szétoszt(X, M, Kicsi, Nagy),
    rendez2(Kicsi, K),
    rendez2(Nagy, N),
    hozzáfűz(K, [X|N], Y).
```

A szétosztás elég magától értetődő:

```prolog
szétoszt(_, [], [], []).
szétoszt(X, [Y|M], K, [Y|N]) :- X =< Y, szétoszt(X, M, K, N).
szétoszt(X, [Y|M], [Y|K], N) :- X > Y, szétoszt(X, M, K, N).
```

Ez (nagyon hosszú listákra) bizonyíthatóan sokkal kevesebb ellenőrzést végez, mint a beszúrásos rendezés. A fenti program azonban a hozzáfűzések miatt nem igazán hatékony - a következő leckében majd szó lesz a különbség-listákról, amelyek segítségével sokkal gyorsabbá tehető.

### Fésülő rendezés (*merge sort*)

Utolsónak nézzünk meg egy nagyon hasonló algoritmust: itt is két részre bontjuk mindig a listát, azokat rendezzük, majd összefűzzük, de itt nem a szétválasztás a bonyolultabb, hanem az összefűzés, vagy ez esetben az *összefésülés*.

A listát a felénél kettéosztjuk, és mindkét részt rendezzük (rekurzívan), végül a két rendezett listát egy listává fésüljük össze:

```prolog
rendez3([], []).
rendez3([X], [X]).
rendez3(X, Y) :-
    kettéoszt(X, X1, X2),
    rendez3(X1, Y1),
    rendez3(X2, Y2),
    összefésül(Y1, Y2, Y).

kettéoszt([], [], []).
kettéoszt([X], [X], []).
kettéoszt([X,Y|M], [X|M1], [Y|M2]) :- kettéoszt(M, M1, M2).
```

Az összefésülésnél a két lista elemét összehasonlítjuk, és a megfelelőt berakjuk a maradékok összefűzéséből kapott listába:

```prolog
összefésül([X|Mx], [Y|My], [X|M]) :- X =< Y, összefésül(Mx, [Y|My], M).
összefésül([X|Mx], [Y|My], [Y|M]) :- X > Y, összefésül([X|Mx], My, M).
összefésül(X, [], X).
összefésül([], Y, Y).
```

Itt érdekes módon azt tapasztaljuk, hogy a (helyes) eredményt végtelen sokszor megkapjuk. Ez azért van, mert a `rendez3` harmadik szabálya az egyelemű listákra is meghívódhat. Ezt ki lehetne védeni azzal, hogy megköveteljük, hogy legyen legalább két eleme:

```prolog
rendez3(X, Y) :-
    X = [_,_|_], Y = [_,_|_],
    kettéoszt(X, X1, X2),
    rendez3(X1, Y1),
    rendez3(X2, Y2),
    összefésül(Y1, Y2, Y).
```

... de ez nem a legelegánsabb. (Hasonlóan, az `összefésül` utolsó két szabálya közt is van átfedés.) Nincs valami jobb mód erre?

## Vágás

Egy szabály törzsében *vágást* eszközölhetünk a `!` segítségével. Ez azt mondja, hogy ha már idáig eljutottunk, akkor vagy sikerül teljesíteni a törzs maradék részét, vagy ha nem, akkor úgy vesszük, hogy ezzel a fejjel való egyesítés sikertelen volt, nem próbálunk a `!` előtti kifejezésekre visszamenni, vagy az azonos fejhez tartozó esetleges többi szabályt megnézni.

Nézzük meg például az alábbi szabályokat, ahol `A`, `B`, `C` stb. kifejezéseket jelölnek:
```prolog
C :- P, Q, R, !, S, T, U.
C :- V.

A :- B, C, D.
```

Ekkor ha az
```
?- A.
```
kérdést feltesszük, a következő fog történni:

1. Megnézi, hogy `B` teljesül-e (tegyük fel, hogy igen).
2. Megnézi, hogy `P`, `Q`, `R` teljesülnek-e (tegyük fel, hogy igen).
3. Megnézi, hogy `S`, `T`, `U` teljesülnek-e. Tegyük fel, hogy az `S` és `T` teljesül, de az `U` nem. Ekkor szokás szerint visszamegy a `T`-re, és megkeresi annak egy másik megoldását stb.
4. Ha nem sikerült az `S`, `T`, `U`-t teljesíteni, akkor - a vágás miatt - már nem megy vissza, hogy az `R` egy másik megoldását keresse, és nem próbálkozik a `C`-hez tartozó második szabállyal sem, hanem egy szinttel feljebb megy, és a `B`-hez keres egy másik megoldást.

### Összefésülés hatékonyabban

Nézzük meg, hogyan lehet ezzel feljavítani az összefésülést!

```prolog
összefésül([X|Mx], [Y|My], [X|M]) :- X =< Y, !, összefésül(Mx, [Y|My], M).
összefésül([X|Mx], [Y|My], [Y|M]) :- X > Y, !, összefésül([X|Mx], My, M).
összefésül(X, [], X) :- !.
összefésül([], Y, Y) :- !.
```

A vágások a programot hatékonyabbá teszik: ha az első szabályban láttuk, hogy `X =< Y`, már nem kell ellenőrizni a többi szabályt stb. (Az utolsó sorban a vágás felesleges, csak a szimmetria kedvéért került bele.)

Hasonlóan, a `rendez3` második szabályában:
```prolog
rendez3([X], [X]) :- !.
```
Ezekkel a módosításokkal már egyértelmű (*determinisztikus*) lesz a megoldás.

A vágások nem változtatják meg a program értelmét (tehát, hogy milyen kifejezésekre lesz igaz), csak a hatékonyságát.

### Maximum

Nézzünk egy másik példát! Az alábbi szabály két szám közül kiválasztja a nagyobbikat:

```prolog
max1(X, Y, X) :- X >= Y.
max1(X, Y, Y) :- X < Y.
```

A fenti módon ez átírható így:

```prolog
max2(X, Y, X) :- X >= Y, !.
max2(X, Y, Y) :- X < Y.
```

Felmerül akkor, hogy a második szabályban szükség van-e egyáltalán az `X < Y` összehasonlításra, hiszen csak akkor juthatunk oda, ha az `X >= Y` nem teljesült. A program akkor is működni látszik, ha elhagyjuk:

```prolog
max3(X, Y, X) :- X >= Y, !.
max3(_, Y, Y).
```

Teszteljük egy kicsit!

```prolog
?- max3(3, 5, X).
X = 5
?- max3(4, 2, X).
X = 4
?- max3(1, 5, 5).
true
?- max3(5, 1, 1). % Ajjaj...
true
```

Hoppá! Mi történt?

Mivel az utolsó példában mindhárom argumentumnak van értéke, és az első és harmadik nem azonos, az első szabállyal nem is próbálkozik, hanem rögtön a másodikra megy, ami pedig most, hogy kivettük a feltételt, teljesül.

A `max3` változatban megváltozott a program deklaratív jelentése, hiszen egy olyan tényt tartalmaz, ami magában nézve nem igaz. Ez általában kerülendő, bár a hatékonysághoz néha szükséges. Ha ilyen gondolatmenetet alkalmazunk, nagyon óvatosnak kell lenni - itt pl. biztosnak kell lennünk benne, hogy a harmadik argumentum mindig változó.

### Hozzáadás halmazhoz

Egy további példaként nézzük meg az alábbi szabályt:

```prolog
% hozzáad(+X, +Halmaz, -Eredmény).
% Hozzáadja `X`-et a `Halmaz` listához,
% de csak akkor, ha az még nem tartalmazta.
hozzáad(X, L, L) :- tartalmaz(X, L), !.
hozzáad(X, L, [X|L]).
```

A vágás nélkül itt a második szabályt nehezebb lenne megfogalmazni, kéne hozzá a 3. leckéhez tartozó projektben látott `nemtartalmaz` szabály, és következésképp a hatékonyságból is vesztene.

Cserébe itt is problémába ütközünk, ha a harmadik argumentum nem változó:
```
?- hozzáad(b, [a,b], [b,a,b]).
true
```
Ezt elkerülendő, érdemes a dokumentációval egyértelművé tenni a használatot. A fenti programhoz tartozó megjegyzés is ilyen szellemben íródott. Az első sorában egy egyezményes jelölésrendszert használ, melyben minden argumentum háromféle lehet:

- `+X` : kell, hogy legyen értéke
- `?X` : lehet értéke, de nem muszáj
- `-X` : csak változó lehet

A `help` is ezeket használja. Például:

```
?- help(>).
+Expr1 > +Expr2
...
?- help(flatten). % lapít (3. lecke 10. feladat)
flatten(+NestedList, -FlatList)
...
?- help(member). % tartalmaz
member(?Elem, ?List)
...
```
Az első esetben mindkét kifejezésnek ismertnek kell lennie; a másodikban a lapítandó lista ismert, és a lapított verzió csak változó lehet; a harmadikban pedig mind a lista, mind a benne tartalmazandó elem lehet változó vagy ismert is.

### Feladatok

1. Nézzük meg az alábbi szabályokat!

       p(1).
       p(2) :- !.
       p(3).
       
   Mit lesz az összes válasz az alábbi kérdésekre:
   
       ?- p(X).
       ?- p(X), p(Y).
       ?- p(X), !, p(Y).

2. Írjátok át hatékonyabbra a `szétoszt` szabályt vágások segítségével!

## Tagadás

Hogyan tudjuk megfogalmazni azt, hogy "Csilla szeret minden állatot, kivéve a pókokat"?

```prolog
szereti(csilla, X) :- pók(X), !, fail.
szereti(csilla, X) :- állat(X).
```

(Általában ilyenkor a `false` helyett a `fail`-t szokás használni, de a kettő jelentése azonos.)

Próbáljuk ki!

```prolog
állat(tarantula).
állat(denevér).
pók(tarantula).

?- szereti(csilla, tarantula).
false
?- szereti(csilla, denevér).
true
```

Ez annyira hasznos, hogy érdemes bevezetni, mint tagadást:

```prolog
nem(P) :- P, !, fail.
nem(_).
```

Figyeljük meg, hogy itt valami olyat csináltunk, amit eddig még soha: egy változót (`P`) a törzsben magában használtunk. Ez feltételezi, hogy a `P`-nek kiszámolható az igazságértéke. Például a fenti példát átírva:

```prolog
szereti(csilla, X) :- állat(X), nem(pók(X)).
```

Itt a `P` értéke a `pók(X)` struktúra, és a `nem` törzsében levő `P` kiértékelésekor ezt mint megoldandó célkifejezést értelmezi.

### A "zárt világ" feltétel

A tagadásnak ez a módja nem mindig intuitív. Egy így tagadott kifejezés akkor lesz igaz, ha a kifejezés nem bizonyítható.

Mi történik, ha az előző programban felcseréljük a törzs két tagját?

```prolog
szereti(csilla, X) :- nem(pók(X)), állat(X).
```

Érdekes módon Csilla most már a denevéreket sem szereti! Miért? Azért, mert a `nem(pók(X))`-ben az `X` egy változó, tehát a `pók(X)` bizonyítható az `X = tarantula` helyettesítéssel, így a `nem(pók(X))` nem teljesül. Az érték nélküli argumentumokkal tehát vigyázni kell.

Hasonlóan furcsa lehet, hogy

```
?- nem(állat(kutya)).
true
```

... de persze helyes, hiszen a programban levő szabályok alapján nem bizonyítható, hogy a kutya állat.

Ezek miatt a `nem` (vagy az angol `not`) helyett a `\+` operátort szokás használni, melynek definíciója ugyanaz, csak az elnevezés kevésbé félrevezető:

```prolog
szereti(csilla, X) :- állat(X), \+ pók(X).
```

### Feladatok

1. Milyen kérdéssel lehet a `Jelöltek` listából kiválasztani azokat, akik nem szerepelnek a `Kiesettek` listában? Használjátok a `tartalmaz` szabályt és a tagadást!
2. Készítsetek szabályt, ami két halmaz különbségét képzi! (A halmaz itt egy olyan lista, amiben minden elem egyszer fordul elő.)

       ?- különbség([a,b,c,d], [f,d,b,e], X).
       X = [a, c]

3. Készítsetek szabályt, ami kiválogatja egy listából azokat a kifejezéseket, amelyek egy adott másik kifejezéssel egyesíthetőek!

       ?- egyesíthető([X,b,t(Y)], t(a), L).
       L = [X, t(Y)]

   Figyeljetek arra, hogy az `X` és `Y` ne kapjon ezáltal értéket!

## Megjegyzések

Ez a dokumentum az alábbi könyv 5. és 9.1. fejezete alapján készült:

I. Bratko: *Prolog Programming for Artificial Intelligence*, 4th Ed., Pearson, 2011.

Az összefésüléses példa az alábbi könyv 11. fejezetéből származik:

L. Sterling, E. Shapiro: *The Art of Prolog*, 2nd Ed., MIT Press, 1994.

Rendezési algoritmusokból nincs hiány. Bizonyítható, hogy a gyorsrendezés, a fésülő rendezés (és még sok másik) a lehető leghatékonyabb ... kivéve egy-két furcsa módszert:

1. Vágjunk (száraz) spagettitésztákat az egyes számoknak megfelelő hosszokra, majd marokra fogva tegyük le az asztalra. Ezután egy kartonlapot rátéve mindig sorban vegyük ki azt, ami hozzáér a laphoz, és így egy csökkenő sorrendezéshez jutunk.
2. Vegyük a lista egy véletlenszerű permutációját, és ellenőrizzük, hogy sorban van-e. Ha igen, készen vagyunk. Ha nem, akkor robbantsuk fel a világot. Feltéve, hogy igaz a párhuzamos univerzumok elmélete, csak az az univerzum fog megmaradni, amelyikben a véletlen permutáció pont a helyes sorbarendezés volt.