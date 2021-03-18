# Prolog programozás 2

## Típusok

Most, hogy már van egy első képünk a Prologról, ideje pontosabban megvizsgálni az alkotóelemeit.

### Atomok

Láttuk, hogy vannak "konkrét dolgok", amiket kisbetűvel írunk - ezeket *atom*oknak szokás nevezni. Az atomok neve háromféleképpen képezhető:

1. Betűk, számok és az alsóvonás (`_`) karakter kombinációja, de az első mindig egy kisbetű. Pl.: `nil`, `foo1`, `bar_42`, `baz__`, `ez_1_hosszú_név`.
2. Különleges karakterek (nem betű/szám/alsóvonás) sorozata, pl. `.:.`, `==>`, `+`.
3. Aposztófok közti karaktersorozat, pl. `'Peti'`, `'Abú Tálib'`.

Az első fajtára már sok példát láttunk; a másodikról majd később lesz szó; a harmadikat pedig jellemzően akkor használjuk, ha nagybetűvel kezdődő vagy szóközt tartalmazó nevet szeretnénk adni (ritka).

### Számok

A Prolog egész és valós számokat tud kezelni, a megszokott jelölésekkel. A valós számoknál lehet használni az "exponenciális formát" is, ahol egy `e` betű után a 10 kitevője szerepel, tehát pl.  `1.23e5 = 123000.0` és `4.56e-3 = 0.00456`.

Egy egész és egy valós szám nem ugyanaz még akkor sem, ha ugyanaz az értékük, tehát
```
?- 1.23e5 = 123000.
```
értéke `false`. Van viszont egy matematikai egyenlőségvizsgálat (`=:=`), és arra már teljesül, hogy
```
?- 1.23e5 =:= 123000.
```
(A számok közti műveletekről majd később lesz még szó.)

### Változók

Ahogy láttuk, a "határozatlan dolgok" (*változók*) nagybetűvel vagy alsóvonással kezdődnek, pl. `X`, `_`, `_Y`, `_z`.

Ezek közül a `_` változónév különleges: minden alkalommal egy független változót jelöl. Tegyük fel például, hogy egy sakkprogramban az állás
```prolog
tábla(világos, futó, c, 3).
```
alakú tényekkel van leírva. Ekkor világos összes bábuját lekérdezhetjük úgy, hogy
```prolog
?- tábla(világos, X, _, _).
```
De ugyanez nem működne, ha más azonos változónevet adnánk meg, pl.
```prolog
?- tábla(világos, X, _hely, _hely).
```

A többi változó egy-egy szabályon belül mindig ugyanazt jelöli, de a szabályok közt már azok is függetlenek, pl.
```prolog
ős(X, Z) :- szülő(X, Z).           % a bal- és jobboldali X,Z ugyanaz
ős(X, Z) :- szülő(X, Y), ős(Y, Z). % de itt már egy másik X és Z van
```

### Összetett struktúrák

A struktúra vagy *funktor* értelmileg összetartozó dolgokat kapcsol össze. A tények és a szabályok feje is egy-egy funktor, tehát az alakja már ismerős: valamilyen név (egy atom), és utána zárójelek közt, vesszőkkel elválasztva a hozzátartozó dolgok (ezek az *argumentumok*, a számuk pedig a funktor *aritása*).

Egy kétdimenziós pontot leírhatunk egy `pont(X, Y)` struktúrával. Ha ezután egy háromszöget akarunk létrehozni, akkor azt 6 koordináta helyett leírhatjuk 3 ponttal: `háromszög(P1, P2, P3)` - ez egy újabb struktúra. Például
```prolog
?- P1 = pont(1, 0), P2 = pont(0, 2), P3 = pont(-1, 0), T = háromszög(P1, P2, P3).
```
esetén a `T` értéke `háromszög(pont(1, 0), pont(0, 2), pont(-1, 0))` lesz.

A különböző aritású funktorok különbözőnek számítanak, pl. lehet készíteni egy 3D `pont(X, Y, Z)` funktort is, illetve azonos nevű, de eltérő aritású tények/szabályok is megférnek egymás mellett.

### Feladatok

1. Milyen típusúak az alábbi kifejezések? (atom, szám, változó, struktúra)
	- `Foo`
	- `foo`
	- `'Foo'`
	- `_foo`
	- `'Foo bar baz'`
	- `foo(bar, baz)`
	- `42`
	- `15(X, Y)`
	- `+(bal, jobb)`
	- `három(Kis(Cica))`
2. Milyen struktúrával lehetne jól leírni egy téglalapot? Egy négyzetet? Egy kört?

## Egyesítés

Két kifejezés egyesíthető, ha azonosak, vagy ha a bennük levő változókat be lehet úgy állítani, hogy azonossá váljanak. Például a `dátum(Év, Hónap, 22)` és a `dátum(X, március, Y)` kifejezések egyesíthetőek, ezt az `=` segítségével tudjuk ellenőrizni:
```prolog
?- dátum(Év, Hónap, 22) = dátum(X, március, Y).
Hónap = március,
X = Év,
Y = 22
```
(Mostantól az egyszerűség kedvéért a kérdésekre kapott eredményeket egyszerűen a kérdés alá fogom írni.)

Ugyanakkor pl. a `dátum(Év, Hónap, 21)` és `dátum(X, Y, 22)` kifejezések nem egyesíthetőek, mivel a harmadik argumentum különböző; a `dátum(X, Y, Z)` és `pont(X, Y, Z)` kifejezések pedig azért nem, mert más a legkülső ("elsődleges") funktoruk.

Valójában a fenti egyenlőség végtelen sok más módon is kielégíthető, pl.
```prolog
Év = 1982,
Hónap = március,
X = 1982,
Y = 22
```
... de ezek kevésbé általánosak. Az egyesítés mindig a legáltalánosabbat adja.

A pontos szabályok a következők:

1. Ha `S` és `T` konstansok (tehát atomok vagy számok), akkor csak akkor egyesíthetőek, ha azonosak.
2. Ha `S` változó, akkor a két kifejezés egyesíthető, és innentől kezdve `S` "értéke" `T` lesz. (Fordítva hasonlóan.)
3. Ha `S` és `T` is struktúrák, akkor pontosan akkor egyesíthetőek, ha
	- megegyezik az elsődleges (legkülső) funktoruk
	- ugyanannyi az aritásuk
	- minden argumentumuk páronként egyesíthető (az ezekben szereplő változók ebben a rekurzív egyesítésben kaphatnak értéket)

Például a
```prolog
háromszög(pont(1, 1), A, pont(2, 3)) = háromszög(X, pont(4, Y), pont(2, Z))
```
egyesítés az alábbi lépésekből áll:

1. `háromszög = háromszög`
2. `pont(1, 1) = X` (itt az `X` megkapja a `pont(1, 1)` értéket)
3. `A = pont(4, Y)` (itt az `A` megkapja a `pont(4, Y)` értéket)
4. `pont(2, 3) = pont(2, Z)`
	1. `pont = pont`
	2. `2 = 2`
	3. `3 = Z` (itt a `Z` megkapja a `3` értéket)

### Egyesítés funktoron belül
	
Az egyesítést ki lehet használni egy funktoron belül is. Egy szakasz függőlegességét pl. megadhatjuk így:
```prolog
függőleges(szakasz(pont(X, _), pont(X, _))).
```

Most feltehetünk mindenféle érdekes kérdést:
```prolog
?- függőleges(szakasz(pont(1, 1), pont(1, 2))).
true
?- függőleges(szakasz(pont(1, 1), pont(2, Y))).
false
?- függőleges(szakasz(pont(1, 1), pont(X, 2))).
X = 1
?- függőleges(szakasz(pont(X, 3), P)).
P = pont(X, _)
```
(Az utolsónál a rendszer az érdektelen y-koordinátára egy automatikusan generált nevet fog adni a `_` helyett, pl. `_123`.)

### Feladatok

1. Definiáljátok szakaszokra a vízszintességet is, majd döntsétek el a segítségével, hogy van-e olyan szakasz, ami egyszerre vízszintes és függőleges!
2. Egyesíthetőek-e az alábbi kifejezéspárok? Ha igen, milyen értéke lesz a változóknak?
	- `pont(A, B) = pont(1, 2)`
	- `pont(A, B) = pont(X, Y, Z)`
	- `plusz(2, 2) = 4`
	- `+(2, D) = +(E, 2)`
	- `háromszög(pont(-1, 0), P2, P3) = háromszög(P1, pont(1, 0), (pont(0, Y))`
3. Az előző feladat végén a háromszögek milyen családját írtuk le?
4. Készítsetek egy szabályt, ami eldönti, hogy egy téglalap oldalai a tengelyekkel párhuzamosak-e!

## Kétféle olvasat

Egy `P :- Q, R.` alakú szabályt kétféleképpen lehet értelmezni:

1. Leíró (*deklaratív*) olvasatok:
	- `P` igaz akkor, ha `Q` és `R` igaz.
	- `Q` és `R`-ből következik `P`.
2. Működés szerinti (*procedurális*) olvasatok:
	- Ahhoz, hogy megoldjuk `P`-t, *először* megoldjuk `Q`-t, és *aztán* megoldjuk `R`-et.
	- Ahhoz, hogy kielégítsük `P`-t, *először* kielégítjük `Q`-t és *aztán* kielégítjük `R`-et.

A legfontosabb különbség az, hogy a procedurális olvasatokban számít a kifejezések sorrendje.

### Logikai vagy

A deklaratív olvasat pusztán logika. A kifejezéseket eddig mindig vesszővel (`,`) kapcsoltuk össze, ami logikai *és*t jelent. A logikai (megengedő) *vagy*ot a pontosvessző (`;`) jelöli. A vesszőnek van elsőbbsége, tehát a `P :- Q, R; S, T.` kifejezést úgy értelmezzük, mintha `P :- (Q, R); (S, T).` lenne. A pontosvessző helyett mindig írhatunk külön szabályokat, és fordítva, pl. az
```prolog
ős(X, Z) :- szülő(X, Z).
ős(X, Z) :- szülő(X, Y), ős(Y, Z).
```
szabályt írhattuk volna egy sorban is:
```prolog
ős(X, Z) :- szülő(X, Z); szülő(X, Y), ős(Y, Z).
```
De általában a külön szabályokba írt változat jobban olvasható.

### Nyomkövetés

Ahhoz, hogy jobban megértsük a procedurális olvasatot, kövessük végig, hogy mit csinál a rendszer egy egyszerű feladat megoldásakor! Legyen a program a következő:

```prolog
nagy(medve).            % 1
nagy(elefánt).          % 2
kicsi(macska).          % 3
barna(medve).           % 4
fekete(macska).         % 5
szürke(elefánt).        % 6
sötét(Z) :- fekete(Z).  % 7
sötét(Z) :- barna(Z).   % 8
```

A kérdés pedig: `?- sötét(X), nagy(X).` Tehát "Mi az ami sötét színű és nagy?"

A megoldáshoz vezető lépések:

1. A teljesítendő célok listája `sötét(X), nagy(X)`.
2. Végigmegyünk a programon az elejétől kezdve, hogy találunk-e a `sötét(X)`-el egyesíthető szabály-fejet (vagy tényt, hiszen a tények tulajdonképpen szabályok, amelyeknek a törzse `true`). A 7-es sor az első ilyen. Az egyesítés miatt `Z = X`, a `sötét(X)`-et helyettesítjük a szabály törzsével, az új cél-lista `fekete(X), nagy(X)`.
3. Végigmegyünk a programon az elejétől kezdve, hogy találunk-e a `fekete(X)`-el egyesíthető szabály-fejet. Az 5-ös sor az első ilyen. Az egyesítés miatt `X = macska`, és mivel az 5-ös sornak nincsen törzse, az új cél-lista `nagy(macska)`.
4. Most a `nagy(macska)`-val egyesíthető szabályt keresünk, de ilyen nincs.
	- Visszalépünk egyet, és az `X` változót ismét szabaddá tesszük. Keresünk egy újabb egyezést a `fekete(X)`-el, onnan, ahol legutóbb abbahagytuk (5-ös sor után), de ilyen sincsen.
	- Visszalépünk még egyet, és újabb egyezést keresünk a `sötét(X)`-el, onnan kezdve, ahol legutóbb abbahagytuk (7-es sor után). Az első ilyen a 8. sorban van. Az egyesítés miatt `Z = X`, a `sötét(X)`-et helyettesítjük a törzzsel, az új cél-lista tehát `barna(X), nagy(X)`.
5. A program elejétől keresünk a `barna(X)`-el egyesíthető szabályt. A 4-es sor az első ilyen. Az egyesítés miatt `X = medve`, és nincs törzs, tehát az új cél-lista `nagy(medve)`.
6. A program elejétől keresünk a `nagy(medve)`-vel egyesíthető szabályt. Rögtön az első sorban meg is találjuk; nincs törzse, és így a cél-listánk elfogyott, készen vagyunk! Az `X` értéke tehát `medve`.

A program végigkövetését a Prolog rendszer is lehetővé teszi a `trace` segítségével:
```prolog
?- trace, sötét(X), nagy(X).
```

Ez most minden egyes lépést kiír, és megkérdezi, mit csináljon. A második *step into* (belelépés) gombot nyomogatva végignézhetjük, mi történik, és közben a programon is mutatja, hogy épp hol tart. Ha be akarjuk fejezni, az első *continue* (folytatás) gomb megnyomásával a program további nyomkövetés nélkül lefut, illetve az utolsó *abort* (megszakítás) gomb hatására azonnal leáll. Az egyes lépések a következő típusúak lehetnek:

- Call (fekete): egy új cél keresése indul
- Redo (sárga): újra próbálkozik egy másik egyesítéssel
- Fail (piros): a cél kielégítése sikertelen
- Exit (zöld): a cél kielégítése sikeres

Itt érdemes még megjegyezni, hogy a beépített funkciókról a `help` segítségével lehet információt kapni (angolul persze), pl. `?- help(trace).` Ha nem tudjuk a nevét annak, amit keresünk, az `apropos` segíthet, ez magában a leírásban keres, pl. az `?- apropos(minimum).` kérdésre megkapjuk azokat a beépített szabályokat, amelyek leírásában szerepel a "minimum" - többek közt a `min/2`-t, tehát a 2 aritású `min` szabályt, amiről további információkat a `?- help(min).` vagy `?- help(min/2).` kérdésekkel kaphatunk.

### Feladatok

1. Írjátok újra az alábbi programot pontosvessző nélkül!

       kiolvas(Szám, Szó) :-
           Szám = 1, Szó = egy;
           Szám = 2, Szó = kettő;
           Szám = 3, Szó = három.
2. Mit ad az

       f(1, egy).
       f(s(1), kettő).
       f(s(s(1)), három).
       f(s(s(s(X))), N) :- f(X, N).
       
   program az alábbi kérdésekre? Ellenőrizzétek a számítógépen!
   - `?- f(s(1), A).`
   - `?- f(s(s(1)), kettő).`
   - `?- f(s(s(s(s(s(s(1)))))), C).`
   - `?- f(D, három).`
3. Vezessétek végig a `?- nagy(X), sötét(X)` kérdés megoldását!
4. A `trace`-t be lehet tenni egy szabály törzsébe is, és akkor onnantól kapcsolódik be a nyomkövetés. Írjátok át a 7. sort erre: `sötét(Z) :- trace, fekete(Z).` és így nézzétek meg a `?- sötét(X), nagy(X).` kérdést! Amikor belép a nyomkövetésbe, nyomjátok meg a *continue* gombot - mi történik? Mi a helyzet akkor, ha a 8. sorhoz is hozzáadjátok a `trace`-t, és újra kipróbáljátok?

## A sorrend fontossága

Ha nem vigyázunk, könnyen írhatunk végtelen rekurziót. Ez a legtisztább formájában úgy néz ki, hogy
```prolog
p :- p.
```
Ha most kiértékeljük a `?- p.` kérdést, akkor a gép csak dolgozik és dolgozik, és nem áll le, csak az `Abort` gomb megnyomásával lehet lelőni. (Bonyolultabb esetekben van, hogy egy idő után egy hibaüzenetet kapunk, hogy "stack limit exceeded", ami lényegében azt jelenti, hogy a rekurzió túl mély lett.)

Az érdekes az, hogy egy program, ami deklaratív olvasatban helyes, vezethet végtelen rekurzióra (tehát a procedurális olvasat szerint hibás). Nézzük meg megint az `ős` szabályt!
```prolog
ős(X, Z) :- szülő(X, Z).
ős(X, Z) :- szülő(X, Y), ős(Y, Z).
```
Itt két sorrendről beszélhetünk: a szabályok sorrendjéről, és a szabályon belüli kifejezések sorrendjéről. Vizsgáljuk meg az összes verziót!
```prolog
% A múltkori családfa egy része
szülő(ámna, mohamed).
szülő(abdulla, mohamed).
szülő(mohamed, zajnab).
szülő(mohamed, fátima).
szülő(fátima, huszajn).

% Eredeti
ős1(X, Z) :- szülő(X, Z).
ős1(X, Z) :- szülő(X, Y), ős1(Y, Z).

% Szabályok felcserélve
ős2(X, Z) :- szülő(X, Y), ős2(Y, Z).
ős2(X, Z) :- szülő(X, Z).

% Kifejezések felcserélve
ős3(X, Z) :- szülő(X, Z).
ős3(X, Z) :- ős3(Y, Z), szülő(X, Y).

% Mindkettő felcserélve
ős4(X, Z) :- ős4(Y, Z), szülő(X, Y).
ős4(X, Z) :- szülő(X, Z).
```
A deklaratív olvasat a cserék során lényegesen nem változik, az mindig jó lesz. Mi a helyzet a procedurális olvasattal?

Az `ős1` a már ismert verzió:
```prolog
?- trace, ős1(abdulla, fátima).
Call:ős1(abdulla, fátima)
 Call:szülő(abdulla, fátima)
 Fail:szülő(abdulla, fátima)
Redo:ős1(abdulla, fátima)
 Call:szülő(abdulla, Y)
 Exit:szülő(abdulla, mohamed)
 Call:ős1(mohamed, fátima)
  Call:szülő(mohamed, fátima)
  Exit:szülő(mohamed, fátima)
 Exit:ős1(mohamed, fátima)
Exit:ős1(abdulla, fátima)
true
```

Az `ős2` esetében először mindig nem-szülő őst próbál keresni:
```prolog
?- trace, ős2(abdulla, fátima).
Call:ős2(abdulla, fátima)
 Call:szülő(abdulla, Y1)
 Exit:szülő(abdulla, mohamed)
 Call:ős2(mohamed, fátima)
  Call:szülő(mohamed, Y2)
  Exit:szülő(mohamed, zajnab)
  Call:ős2(zajnab, fátima)
   Call:szülő(zajnab, Y3)
   Fail:szülő(zajnab, Y3)
  Redo:ős2(zajnab, fátima)
   Call:szülő(zajnab, fátima)
   Fail:szülő(zajnab, fátima)
  Fail:ős2(zajnab, fátima)
  Redo:szülő(mohamed, Y2)
  Exit:szülő(mohamed, fátima)
  Call:ős2(fátima, fátima)
   Call:szülő(fátima, Y3)
   Exit:szülő(fátima, huszajn)
   Call:ős2(huszajn, fátima)
    Call:szülő(huszajn, Y4)
    Fail:szülő(huszajn, Y4)
   Redo:ős2(huszajn, fátima)
    Call:szülő(huszajn, fátima)
    Fail:szülő(huszajn, fátima)
   Fail:ős2(huszajn, fátima)
  Redo:ős2(fátima, fátima)
   Call:szülő(fátima, fátima)
   Fail:szülő(fátima, fátima)
  Fail:ős2(fátima, fátima)
 Redo:ős2(mohamed, fátima)
  Call:szülő(mohamed, fátima)
  Exit:szülő(mohamed, fátima)
 Exit:ős2(mohamed, fátima)
Exit:ős2(abdulla, fátima)
true
```

Az `ős3` a rekurzív szabálynál először az ősséget ellenőrzi, csak aztán a szülőséget:
```prolog
?- trace, ős3(abdulla, fátima).
Call:ős3(abdulla, fátima)
 Call:szülő(abdulla, fátima)
 Fail:szülő(abdulla, fátima)
Redo:ős3(abdulla, fátima)
 Call:ős3(X, fátima)
  Call:szülő(X, fátima)
  Exit:szülő(mohamed, fátima)
 Exit:ős3(mohamed, fátima)
 Call:szülő(abdulla, mohamed)
 Exit:szülő(abdulla, mohamed)
Exit:ős3(abdulla, fátima)
true
```

Az `ős4`-nél végtelen rekurzió jön létre:
```prolog
?- trace, ős4(abdulla, fátima).
Call:ős4(abdulla, fátima)
 Call:ős4(X1, fátima)
  Call:ős4(X2, fátima)
   Call:ős4(X3, fátima)
    Call:ős4(X4, fátima)
     ...
```

Egy jó ökölszabály, hogy érdemes az egyszerűbb szabályokat előre tenni (`ős1` és `ős3`), és a szabályokon belül is az egyszerűbb kifejezések jöjjenek előbb (`ős1`).

### Feladatok

1. Menjetek végig a fenti nyomkövetéseken soronként (ha esetleg megspóroltátok volna), és legyetek biztosak benne, hogy minden világos!
2. Az `ős3` esetében, ha további megoldást keresünk a `Next` gombbal (ami itt az "Abdulla őse Fátimának" állítás egy másik bizonyítását jelentené), akkor megint végtelen rekurzióba kerül a program. Miért? Nézzétek meg nyomkövetéssel!

## Megjegyzések

Ez a dokumentum az alábbi könyv 2. fejezete alapján készült:

I. Bratko: *Prolog Programming for Artificial Intelligence*, 4th Ed., Pearson, 2011.

A programozók előszeretettel használják a `foo`, `bar` és `baz` neveket, amikor valamilyen példát mutatnak, ahol maga a név nem fontos. Hasonlóan, ha egy példában egy egész számot kell választani, ez leggyakrabban a `42` (a *Galaxis útikalauz stopposoknak* nyomán). Ilyen és hasonló programozó-szubkultúrával kapcsolatos érdekességekről rengeteget lehet olvasni a [*Zsargon fájl*ban](http://www.catb.org/jargon/html/), nagyon szórakoztató olvasmány. (Sajnos csak angolul, de sokszor nem is igazán lehetne lefordítani.) A fő része a Glossary (szószedet), ajánlom pl. a programhibákkal kapcsolatos szócikkeket (*bug*, *heisenbug*, *mandelbug*, *schroedinbug* stb.).

Ha az előző bekezdés végén levő pont-zárójel-pont kombináció szemet szúrt, akkor ajánlom a programozók írásstílusáról szóló részt (Hacker Writing Style) is. :)
