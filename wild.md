# Prolog mélyvíz

Most, hogy már tisztában vagyunk a Prolog alapjaival, nézzük meg, hogy hogy néz ki egy "igazi" Prolog program: az asszociációs struktúrát megvalósító könyvtár SWI Prologban.

Az alábbiakban a [forráskódot](https://github.com/SWI-Prolog/swipl-devel/blob/97a77e76b2013d65cf34e5ea31f6330ca42660f9/library/assoc.pl) *teljesen változatlan formában* közlöm, csak közbeszúrok magyarázatokat.

## Háttér

Mielőtt megnéznénk magát a programot, vizsgáljuk meg, hogy mi is a probléma, amire megoldást ad, és mi ennek az elméleti háttere.

### Asszociációs listák

Programozáskor nagyon gyakran előfordul, hogy adatokat *kulcsokhoz* rendelünk, amelyek szerint később ki akarjuk majd keresni őket. Az a feltételezés, hogy különböző adatokhoz mindig különböző kulcs tartozik. Ilyen kulcs lehet pl. egy bankban a számlaszám, amihez a megfelelő számla adatait rendelik. A kulcs-érték párokat tároló adatstruktúrákat gyakran nevezik *szótáraknak* is, hiszen egy szótárban (enciklopédiában stb.) is egy-egy címszóhoz vannak rendelve a jelentések/magyarázatok.

A szótár legegyszerűbb megvalósítása az *asszociációs lista*, ahol a párokat egy listában tároljuk:

```prolog
empty_assoc([]).

put_assoc(K, A, V, [K-V|A]).

get_assoc(K, [K-V|_], V) :- !.
get_assoc(K, [K1-_|A], V) :- K \= K1, get_assoc(K, A, V).

del_assoc(K, [K-V|A], V, A1) :- !, del_assoc_others(K, A, A1).
del_assoc(K, [K1-V1|A], V, [K1-V1|A1]) :- K \= K1, del_assoc(K, A, V, A1).

del_assoc_others(_, [], []) :- !.
del_assoc_others(K, [K-_|A], A1) :- !, del_assoc_others(K, A, A1).
del_assoc_others(K, [K1-V1|A], [K1-V1|A1]) :- K \= K1, del_assoc_others(K, A, A1).
```

Itt az üres szótár egyszerűen egy üres lista; a `put_assoc` egy asszociációs lista elejére rak be egy kulcs-érték párt; a `get_assoc` megkeresi a listában az első adott kulcsú értéket; a `del_assoc` pedig kitörli ugyanezt. A törlésnél figyelni kell arra, hogy az összes lehetséges előfordulást töröljük, ezért ez mindig a teljes listán végigmegy.

Amíg nincsen nagyon sok adatunk, ez a megoldás elég jól működik. Az egyszerűségnek azonban ára van: mind a keresés, mind a törlés általános esetben az elemek számával arányos. Ezt úgy szokás megfogalmazni, hogy a keresés és törlés komplexitása *O*(*n*), ahol *n* az elemek száma. Ez az *O* (kiolvasva *ordó*) azt mondja, hogy nem biztos, hogy pontosan *n* művelet, lehet hogy *n*+2, vagy *5n*, de ha az *n*-et 100-szor akkorára választom, akkor a műveletigény is körülbelül 100-szor akkorára nő. (Egy *O*(*n*^2)-es algoritmus esetén ilyenkor a műveletigény kb. a 10 000-szeresére változna.)

A következőkben bemutatott módszer olyan, hogy mindhárom művelet (beszúrás, keresés, törlés) egyaránt *O*(log *n*) komplexitású, tehát ha az *n* a 100-szorosára nő, akkor a műveletigény kb. 6-7-szeresére változik. A beszúrás a fenti egyszerű verzióban gyorsabb - *O*(1) -, de a keresés és törlés hatkékonysága miatt érdemesebb az alábbi adatstruktúrákat alkalmazni.

### Bináris keresőfák

Tegyük fel, hogy a kulcsok sorbarendezhetőek (Prologban tetszőleges két kifejezés sorbarendezhető a `@<` operátorral). Ekkor a kulcsokat egy olyan (általában fejjel lefele ábrázolt) fa alakú struktúrába lehet szervezni, ami mindig legfeljebb kétfelé ágazik el (ezért *bináris fának* hívják). Nézzünk egy példát, ahol a kulcsok számok:

```
       8
      / \
     /   \
    /     \
   3      10
  / \       \
 /   \       \
1     6      14
     / \     /
    4   7   13
```

Itt a fa *gyökere* a 8, belső *csúcsok* a 3, 10, 6 és 14, és a fa *levelei* az 1, 4, 7 és 13. Amikor egy csúcsból csak egy ág megy tovább (pl. 10, 14), olyankor is az ág vagy balra, vagy jobbra megy. Egy ilyen fát könnyen le tudunk írni Prologban, például egy `fa` struktúrával, aminek az első argumentuma a kulcs, a második és harmadik pedig a bal- és jobboldali ág (a nem létező ágakat jelölje mondjuk a `-`):

```prolog
fa(8, fa(3, fa(1, -, -),
            fa(6, fa(4, -, -),
                  fa(7, -, -))),
      fa(10, -,
             fa(14, fa(13, -, -),
                    -)))
```

(Egy másik lehetőség, hogy a levelekre egy külön `levél` funktort használunk stb.)

A fenti példában szereplő fának van egy különleges tulajdonsága: egy csúcs alatti baloldali ágon (és az abból kijövő ágakon stb.) minden elem kisebb, a jobboldali ágon pedig mindegyik nagyobb, mint a csúcsban levő érték. Az ilyen tulajdonságú fákat *keresőfának* nevezik.

Ha meg akarunk keresni egy elemet, akkor elindulunk a gyökértől, és aszerint, hogy a keresett kulcs kisebb, vagy nagyobb, balra ill. jobbra megyünk tovább. Ezt addig folytatjuk, amíg meg nem találjuk a keresett elemet, vagy egy levélhez nem érünk. A keresés műveletigénye tehát a fa magasságával arányos. A beszúrásról és törlésről hasonlóan megmutatható, hogy a fa magasságától függ a komplexitásuk. Ha az adatok szépen egyenletesen helyezkednek el, akkor ez hozzávetőlegesen log *n* lesz (2-es alapú logaritmussal).

Sajnos azonban ez nem feltétlenül teljesül - pl. ez is egy keresőfa:

```
          6
         /
        5
       /
      4
     /
    3
   /
  2
 /
1 
```

... de itt a fa magassága az elemek számával azonos.

### AVL-fák

Egy lehetséges megoldása ennek a problémának az, hogy megköveteljük, hogy a fa mindig *kiegyensúlyozott* legyen, tehát minden csúcsnál a bal- és jobb ághoz tartozó részfa magassága legfeljebb 1-el különbözhet. A fenti példában a 8-as alatti két részfa magassága egyaránt 3, a 3-as alatti két részfa magassága 1 és 2, de pl. a 10-es alatti két részfa magassága 0 és 2, tehát ez nem kiegyensúlyozott. Ha a 10-14-13 hármast "átforgatjuk", akkor kiegyensúlyozottá válik:

```
       8
      / \
     /   \
    /     \
   3      13
  / \     / \
 /   \   /   \
1     6 10   14
     / \
    4   7
```

A beszúrás és törlés műveletekbe ilyen jellegű forgatásokat épít be az *AVL-fa*, hogy biztosítja a kiegyensúlyozottságot. Ezt a módszert 1962-ben publikálta két szovjet matematikus, Adelszon-Velszkij és Landisz, az ő vezetéknevükből származik az adastruktúra elnevezése.

Az alább vizsgált program AVL-fát használ a szótár megvalósítására - a pontos részleteket majd útközben megbeszéljük. Kalandra fel!

## A program

```prolog
/*  Part of SWI-Prolog

    Author:        R.A.O'Keefe, L.Damas, V.S.Costa, Glenn Burgess,
                   Jiri Spitz and Jan Wielemaker
    E-mail:        J.Wielemaker@vu.nl
    WWW:           http://www.swi-prolog.org
    Copyright (c)  2004-2018, various people and institutions
    All rights reserved.

    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions
    are met:

    1. Redistributions of source code must retain the above copyright
       notice, this list of conditions and the following disclaimer.

    2. Redistributions in binary form must reproduce the above copyright
       notice, this list of conditions and the following disclaimer in
       the documentation and/or other materials provided with the
       distribution.

    THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
    "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
    LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
    FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
    COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
    INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING,
    BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
    LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
    CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
    LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN
    ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
    POSSIBILITY OF SUCH DAMAGE.
*/
```

A `/*` és `*/` közti rész megjegyzésnek számít, olyan, mint ha minden sor elején lenne egy `%` szimbólum.

```prolog
:- module(assoc,
          [ empty_assoc/1,              % -Assoc
            is_assoc/1,                 % +Assoc
            assoc_to_list/2,            % +Assoc, -Pairs
            assoc_to_keys/2,            % +Assoc, -List
            assoc_to_values/2,          % +Assoc, -List
            gen_assoc/3,                % ?Key, +Assoc, ?Value
            get_assoc/3,                % +Key, +Assoc, ?Value
            get_assoc/5,                % +Key, +Assoc0, ?Val0, ?Assoc, ?Val
            list_to_assoc/2,            % +List, ?Assoc
            map_assoc/2,                % :Goal, +Assoc
            map_assoc/3,                % :Goal, +Assoc0, ?Assoc
            max_assoc/3,                % +Assoc, ?Key, ?Value
            min_assoc/3,                % +Assoc, ?Key, ?Value
            ord_list_to_assoc/2,        % +List, ?Assoc
            put_assoc/4,                % +Key, +Assoc0, +Value, ?Assoc
            del_assoc/4,                % +Key, +Assoc0, ?Value, ?Assoc
            del_min_assoc/4,            % +Assoc0, ?Key, ?Value, ?Assoc
            del_max_assoc/4             % +Assoc0, ?Key, ?Value, ?Assoc
          ]).
```

Az SWI Prologban a programok könyvtárakba vagy *modulokba* vannak szervezve. Itt az `assoc` modul definícióját látjuk: a `module` első argumentuma a modul neve, a második pedig a modul által szolgáltatott szabályok listája (aritásokkal együtt). A megjegyzések azt is mutatják, hogy az egyes argumentumok mit jelentenek, illetve hogy változót vagy értéket várnak (ld. 5. lecke).

A modul definíciójában nem szereplő szabályok "kívülről" (más programfájlokból) nem látszanak. A fenti kezdetleges asszociációs listánál például a `del_assoc_others` egy ilyen lokális szabály lenne, amit a `del_assoc` ugyan használ, de a könyvtárat használó más program már nem lát.

```prolog
:- autoload(library(error),[must_be/2,domain_error/2]).
```

Ez a sor láthatóvá teszi az `error` modul által szolgáltatott `must_be` és `domain_error` szabályokat. Az `autoload` helyett írhatunk `use_module`-t is, a különbség az, hogy az `autoload` a szabályok betöltését csak akkor végzi el, amikor ténylegesen szükség van rájuk, míg a `use_module` rögtön e sor feldolgozásakor.

```prolog
/** <module> Binary associations

Assocs are Key-Value associations implemented as  a balanced binary tree
(AVL tree).

@see            library(pairs), library(rbtrees)
@author         R.A.O'Keefe, L.Damas, V.S.Costa and Jan Wielemaker
*/
```

A `/** ... */` között levő megjegyzések speciális formátumúak. Ezeket egy külön program feldolgozza, és automatikusan generálja a könyvtárhoz tartozó [dokumentációt](https://www.swi-prolog.org/pldoc/doc/_SWI_/library/assoc.pl). A `<module>`, `@see` és `@author` ennek a külső programnak adott instrukciók.

```prolog
:- meta_predicate
    map_assoc(1, ?),
    map_assoc(2, ?, ?).
```

A `meta_predicate` az utána következő, vesszővel elválasztott szabályokat meta-szabályokként deklarálja, tehát ezek olyan szabályok, amelyek más szabályokon operálnak. Az argumentumoknál a `+`, `-` és `?` jelentése ugyanaz, mint a kommenteknél; a számok azt jelölik, hogy ott az argumentum egy szabály, aminek ennyivel kevesebb argumentuma van.

Például a 6. leckében látott `maplist/2` és `maplist/3` is pontosan így deklarálható. A hasonlatosság nem véletlen - a `map_assoc` ugyanúgy egy szabályt fog alkalmazni minden elemre, csak nem egy *lista* minden elemére, hanem egy asszociációs struktúra (AVL-fa) minden elemére.

```prolog
%!  empty_assoc(?Assoc) is semidet.
%
%   Is true if Assoc is the empty association list.

empty_assoc(t).
```

A `%!`-el kezdődő megjegyzések szintén dokumentáció-generálásra valók, és az azt követő szabályra vonatkoznak. A `semidet` egyike a szabályok öt lehetséges kategóriájának:

- `det` (determinisztikus): mindig pontosan egyszer teljesül, pl. `összeg(+L, -Ö)`

- `semidet` (félig determinisztikus): legfeljebb egyszer teljesül, pl. `maximum(+L, -M)` [üres listára sikertelen]

- `multi` (többszörös): legalább egyszer teljesül, pl. `permutáció(+L, -P)`

- `nondet` (nemdeterminisztikus): többször teljesülhet, de lehet sikertelen is, pl. `tartalmaz(?E, ?L)`

- `failure` (sikertelen): sosem teljesül, pl. `fail`

Ezeket mind úgy kell érteni, hogy "ha a dokumentációjának megfelelően adjuk meg a paramétereket".

Visszatérve az AVL-fára, az üres fát itt a `t` atom fogja jelölni (nem a `-`, mint fent a bináris fánál).

```prolog
%!  assoc_to_list(+Assoc, -Pairs) is det.
%
%   Translate Assoc to a list Pairs of Key-Value pairs.  The keys
%   in Pairs are sorted in ascending order.

assoc_to_list(Assoc, List) :-
    assoc_to_list(Assoc, List, []).

assoc_to_list(t(Key,Val,_,L,R), List, Rest) :-
    assoc_to_list(L, List, [Key-Val|More]),
    assoc_to_list(R, More, Rest).
assoc_to_list(t, List, List).
```

Az `assoc_to_list` szabály egy AVL-fából párok listáját hozza létre. Akkumulátoros megoldás (ld. 6. lecke), tehát egy üres akkumulátor paraméterrel meghívja a 3-argumentumú változatot. Az AVL-fa egy csúcsát a `t(K,V,B,L,R)` struktúra írja le, ahol `K` és `V` a kulcs és a hozzá tartozó érték, `B` a kiegyensúlyozáshoz használt szimbólum (`-` ha a két részfa azonos magasságú, `<` ill. `>` ha a baloldali ill. jobboldali magasabb), `L` és `R` pedig a bal- és jobboldali részfa.

```prolog
%!  assoc_to_keys(+Assoc, -Keys) is det.
%
%   True if Keys is the list of keys   in Assoc. The keys are sorted
%   in ascending order.

assoc_to_keys(Assoc, List) :-
    assoc_to_keys(Assoc, List, []).

assoc_to_keys(t(Key,_,_,L,R), List, Rest) :-
    assoc_to_keys(L, List, [Key|More]),
    assoc_to_keys(R, More, Rest).
assoc_to_keys(t, List, List).

%!  assoc_to_values(+Assoc, -Values) is det.
%
%   True if Values is the  list  of   values  in  Assoc.  Values are
%   ordered in ascending  order  of  the   key  to  which  they were
%   associated.  Values may contain duplicates.

assoc_to_values(Assoc, List) :-
    assoc_to_values(Assoc, List, []).

assoc_to_values(t(_,Value,_,L,R), List, Rest) :-
    assoc_to_values(L, List, [Value|More]),
    assoc_to_values(R, More, Rest).
assoc_to_values(t, List, List).
```

Ez a két szabály gyakorlatilag ugyanaz, mint az előző, csak nem kulcs-érték párokat gyűjtenek ki listába, hanem rendre kulcsokat illetve értékeket.

```prolog
%!  is_assoc(+Assoc) is semidet.
%
%   True if Assoc is an association list. This predicate checks
%   that the structure is valid, elements are in order, and tree
%   is balanced to the extent guaranteed by AVL trees.  I.e.,
%   branches of each subtree differ in depth by at most 1.

is_assoc(Assoc) :-
    is_assoc(Assoc, _Min, _Max, _Depth).

is_assoc(t,X,X,0) :- !.
is_assoc(t(K,_,-,t,t),K,K,1) :- !, ground(K).
is_assoc(t(K,_,>,t,t(RK,_,-,t,t)),K,RK,2) :-
    % Ensure right side Key is 'greater' than K
    !, ground((K,RK)), K @< RK.

is_assoc(t(K,_,<,t(LK,_,-,t,t),t),LK,K,2) :-
    % Ensure left side Key is 'less' than K
    !, ground((LK,K)), LK @< K.

is_assoc(t(K,_,B,L,R),Min,Max,Depth) :-
    is_assoc(L,Min,LMax,LDepth),
    is_assoc(R,RMin,Max,RDepth),
    % Ensure Balance matches depth
    compare(Rel,RDepth,LDepth),
    balance(Rel,B),
    % Ensure ordering
    ground((LMax,K,RMin)),
    LMax @< K,
    K @< RMin,
    Depth is max(LDepth, RDepth)+1.

% Private lookup table matching comparison operators to Balance operators used in tree
balance(=,-).
balance(<,<).
balance(>,>).
```

Az `is_assoc` ellenőrzi, hogy a paraméterben kapott kifejezés egy helyes AVL-fa-e. Az első szabály csak meghívja a 4-argumentumú verziót. A többi az 5 lehetséges esetet kezeli:

1. Az *üres fa* egy helyes AVL-fa. (A többi paraméter értéke itt érdektelen, de azért valami értelmesre vannak beállítva.)

2. A *levél* minimuma és maximuma is a levélben szereplő kulcs, a mélysége pedig 1. A `ground` azt ellenőrzi, hogy a kulcsban nem szerepel ismeretlen változó.

3. Ha csak a *baloldali részfa üres*, a jobboldali részfa alatti részfák üresek kell, hogy legyenek, és az egész fa mélysége 2. Ellenőrzi, hogy a kulcsokban nincsenek ismeretlenek, és a jobboldali részfa kulcsa nagyobb, mint a gyökéré.

4. Ha csak a *jobboldali részfa üres*, akkor ugyanez fordítva.

5. Ha *egyik részfa sem üres*, akkor rekurzívan ellenőrzi mindkettőt. A fa minimuma a baloldali részfa minimuma, a maximuma pedig a jobboldali részfa maximuma lesz. A részfák mélységét összehasonlítja a `compare` segítségével, ami az első argumentumban `<`, `=` vagy `>` lesz. Ellenőrzi, hogy ennek megfelel-e a fában szereplő `B` szimbólum (a `balance` az `=`-ből `-` jelet csinál), és hogy a kulcs a baloldali maximum és a jobboldali minimum közé esik. A mélység eggyel több, mint a két részfa mélysége közül a nagyobbik.

```prolog
%!  gen_assoc(?Key, +Assoc, ?Value) is nondet.
%
%   True if Key-Value is an association in Assoc. Enumerates keys in
%   ascending order on backtracking.
%
%   @see get_assoc/3.

gen_assoc(Key, Assoc, Value) :-
    (   ground(Key)
    ->  get_assoc(Key, Assoc, Value)
    ;   gen_assoc_(Key, Assoc, Value)
    ).

gen_assoc_(Key, t(_,_,_,L,_), Val) :-
    gen_assoc_(Key, L, Val).
gen_assoc_(Key, t(Key,Val,_,_,_), Val).
gen_assoc_(Key, t(_,_,_,_,R), Val) :-
    gen_assoc_(Key, R, Val).
```

A `gen_assoc` egy olyan keresés, ami fordítva is tud működni: meg tud keresni egy adott értékhez tartozó kulcsot (természetesen ez nem lesz hatékony), vagy ha az érték sincsen megadva, akkor végigmegy az összes kulcs-érték páron. Ha a kulcs meg van adva, akkor ugyanazt csinálja, mint a `get_assoc`; a tényleges implementáció a `gen_assoc_` szabályban van.

Mivel először a baloldali részfát vizsgálja, utána a gyökérben levő értéket, és végül a jobboldali részfát (*infix* bejárás), a kulcsokat növekvő sorrendben fogja végigvenni.

```prolog
%!  get_assoc(+Key, +Assoc, -Value) is semidet.
%
%   True if Key-Value is an association in Assoc.
%
%   @error type_error(assoc, Assoc) if Assoc is not an association list.

get_assoc(Key, Assoc, Val) :-
    must_be(assoc, Assoc),
    get_assoc_(Key, Assoc, Val).

:- if(current_predicate('$btree_find_node'/5)).
get_assoc_(Key, Tree, Val) :-
    Tree \== t,
    '$btree_find_node'(Key, Tree, 0x010405, Node, =),
    arg(2, Node, Val).
:- else.
get_assoc_(Key, t(K,V,_,L,R), Val) :-
    compare(Rel, Key, K),
    get_assoc(Rel, Key, V, L, R, Val).

get_assoc(=, _, Val, _, _, Val).
get_assoc(<, Key, _, Tree, _, Val) :-
    get_assoc(Key, Tree, Val).
get_assoc(>, Key, _, _, Tree, Val) :-
    get_assoc(Key, Tree, Val).
:- endif.
```

Megadott kulcs alapján keres. A `must_be` ellenőrzi, hogy az `Assoc` változó `assoc` "típusú"-e - ehhez a `has_type` szabályt használja, amit majd még ki kell bővíteni, hogy értelmezze az `assoc` típust (ld. fájl vége).

Az `:- if(X). ... :- else. ... :- endif.` kifejezések a fordítónak szólnak: ha `X` teljesül, akkor az `:- if(X).` utáni, ha nem, akkor az `:- else.` utáni kifejezést kell csak lefordítani. A `current_predicate` azt ellenőrzi, hogy a zárójelben levő szabály létezik-e. Jelen esetben azt nézi meg, hogy a `'$btree_find_node'/5` definiálva van-e; ez egy C programnyelven írt, bináris fában kereső, [nagyon hatékony eljárás](https://github.com/SWI-Prolog/swipl-devel/blob/master/src/pl-btree.c).

A `\==` operátor az `==` tagadása, ami (egyesítés nélküli) azonosságot tesztel, tehát pl. egy változó csak önmagával vagy egy vele már korábban egyesített változóval lesz azonos. A `'$btree_find_node'`-ban szereplő furcsa szám azt kódolja le, hogy a `t(K,V,B,L,R)` struktúrában a kulcs az első, a bal- és jobboldali részfa pedig rendre a negyedik és ötödik argumentum. A `Node` a kulcsot tartalmazó csúcsot adja vissza (ha szerepelt a fában), az utolsó argumentum pedig `=`, `<` vagy `>`, attól függően, hogy megtalálta-e az elemet, vagy pedig új bal- ill. jobb-levélként kéne felvenni a fába.

Amennyiben ez a keresőalgoritmus nem elérhető, az `:- else` után található egy tisztán Prologban írt alternatíva, ami nagyon könnyen érthető.

```prolog
%!  get_assoc(+Key, +Assoc0, ?Val0, ?Assoc, ?Val) is semidet.
%
%   True if Key-Val0 is in Assoc0 and Key-Val is in Assoc.

get_assoc(Key, t(K,V,B,L,R), Val, t(K,NV,B,NL,NR), NVal) :-
    compare(Rel, Key, K),
    get_assoc(Rel, Key, V, L, R, Val, NV, NL, NR, NVal).

get_assoc(=, _, Val, L, R, Val, NVal, L, R, NVal).
get_assoc(<, Key, V, L, R, Val, V, NL, R, NVal) :-
    get_assoc(Key, L, Val, NL, NVal).
get_assoc(>, Key, V, L, R, Val, V, L, NR, NVal) :-
    get_assoc(Key, R, Val, NR, NVal).
```

Ez a változat arra használható, hogy egy már létező kulcshoz tartozó értéket lecseréljünk. A harmadik argumentum a kulcshoz tartozó régi érték, a negyedik az így keletkező AVL-fa, és az utolsó az új érték.

```prolog
%!  list_to_assoc(+Pairs, -Assoc) is det.
%
%   Create an association from a list Pairs of Key-Value pairs. List
%   must not contain duplicate keys.
%
%   @error domain_error(unique_key_pairs, List) if List contains duplicate keys

list_to_assoc(List, Assoc) :-
    (  List = [] -> Assoc = t
    ;  keysort(List, Sorted),
           (  ord_pairs(Sorted)
           -> length(Sorted, N),
              list_to_assoc(N, Sorted, [], _, Assoc)
           ;  domain_error(unique_key_pairs, List)
           )
    ).

list_to_assoc(1, [K-V|More], More, 1, t(K,V,-,t,t)) :- !.
list_to_assoc(2, [K1-V1,K2-V2|More], More, 2, t(K2,V2,<,t(K1,V1,-,t,t),t)) :- !.
list_to_assoc(N, List, More, Depth, t(K,V,Balance,L,R)) :-
    N0 is N - 1,
    RN is N0 div 2,
    Rem is N0 mod 2,
    LN is RN + Rem,
    list_to_assoc(LN, List, [K-V|Upper], LDepth, L),
    list_to_assoc(RN, Upper, More, RDepth, R),
    Depth is LDepth + 1,
    compare(B, RDepth, LDepth), balance(B, Balance).
```

Az `assoc_to_list` fordítottja. A fa hatékony építéséhez először sorbarakja az elemeket a kulcsok szerint a `keysort` szabály segítségével. Az `ord_pairs` később lesz definiálva, azt vizsgálja, hogy az elemek szigorú sorrendben vannak, tehát nincs két azonos sem köztük. Ha ez nem teljesül, akkor a `domain_error` hibát jelez: a `List` nem teljesíti a `unique_key_pairs` feltételt, tehát hogy a kulcsok egyértelműek legyenek.

A fa építését az 5-argumentumú verzió végzi. Az első argumentum az elemek száma, a második és harmadik elem együtt a rendezett elemek különbség-listája, a negyedik a fa mélysége, és az utolsó a készített AVL-fa. Az elemek száma alapján:

1. Ha *1 elem van*, akkor egy 1 mélységű, egy levélből álló fa az eredmény.

2. Ha *2 elem van*, akkor egy 2 mélységű, egy bal-levéllel rendelkező fa az eredmény.

3. Ha *legalább 3 elem van*, akkor rekurzívan elkészít két részfát, amelyek a lista első ill. második felét tartalmazzák. Kicsit pontosabban, a két részfa összesen `N - 1` elemet tárol (mivel 1 a gyökérbe kerül), és ha ez nem páros, akkor a baloldaliban lesz több elem. A különbség-lista itt lesz hasznos: az első `LN` elemből elkészül az `L` fa, és a `List` listából fel nem használt maradékot a `[K-V|Upper]` listával egyesíti. Ezáltal a gyökérhez tartozó kulcs-érték pár és a jobboldali fa építéséhez szükséges `Upper` lista is rögtön adott. Mivel a baloldali részfában van több elem, a mélység eggyel több lesz, mint a baloldali mélység. Végül a bal- és jobb mélység alapján kiszámolja a gyökérhez tartozó `B` értéket (`<`, `-` vagy `>`).

```prolog
%!  ord_list_to_assoc(+Pairs, -Assoc) is det.
%
%   Assoc is created from an ordered list Pairs of Key-Value
%   pairs. The pairs must occur in strictly ascending order of
%   their keys.
%
%   @error domain_error(key_ordered_pairs, List) if pairs are not ordered.

ord_list_to_assoc(Sorted, Assoc) :-
    (  Sorted = [] -> Assoc = t
    ;  (  ord_pairs(Sorted)
           -> length(Sorted, N),
              list_to_assoc(N, Sorted, [], _, Assoc)
           ;  domain_error(key_ordered_pairs, Sorted)
           )
    ).
```

Ugyanez, csak már feltételezi, hogy a lista elemei rendezettek.

```prolog
%!  ord_pairs(+Pairs) is semidet
%
%   True if Pairs is a list of Key-Val pairs strictly ordered by key.

ord_pairs([K-_V|Rest]) :-
    ord_pairs(Rest, K).
ord_pairs([], _K).
ord_pairs([K-_V|Rest], K0) :-
    K0 @< K,
    ord_pairs(Rest, K).
```

Ellenőrzi, hogy a lista elemei szigorú sorrendben vannak-e.

```prolog
%!  map_assoc(:Pred, +Assoc) is semidet.
%
%   True if Pred(Value) is true for all values in Assoc.

map_assoc(Pred, T) :-
    map_assoc_(T, Pred).

map_assoc_(t, _).
map_assoc_(t(_,Val,_,L,R), Pred) :-
    map_assoc_(L, Pred),
    call(Pred, Val),
    map_assoc_(R, Pred).
```

Infix bejárással végigmegy az elemeken, és mindegyikre meghívja az első argumentumban kapott szabályt. A `call(Pred, Val)` a `Val` paramétert még hozzácsapja a `Pred` argumentumaihoz, tehát pl. a `call(tartalmaz(42), L)` megfelel a `tartalmaz(42, L)` hívásnak.

```prolog
%!  map_assoc(:Pred, +Assoc0, ?Assoc) is semidet.
%
%   Map corresponding values. True if Assoc is Assoc0 with Pred
%   applied to all corresponding pairs of of values.

map_assoc(Pred, T0, T) :-
    map_assoc_(T0, Pred, T).

map_assoc_(t, _, t).
map_assoc_(t(Key,Val,B,L0,R0), Pred, t(Key,Ans,B,L1,R1)) :-
    map_assoc_(L0, Pred, L1),
    call(Pred, Val, Ans),
    map_assoc_(R0, Pred, R1).
```

Ez a változat két argumentumot ad a kapott szabályhoz: az első (`Val`) az éppen vizsgált érték, mint az előző verzióban, a második (`Ans`) pedig egy változó, amire az adott elemet lecserélve egy új fát épít. Ezzel tehát lehet olyat csinálni, hogy minden értéket a fában négyzetre emelünk stb.

```prolog
%!  max_assoc(+Assoc, -Key, -Value) is semidet.
%
%   True if Key-Value is in Assoc and Key is the largest key.

max_assoc(t(K,V,_,_,R), Key, Val) :-
    max_assoc(R, K, V, Key, Val).

max_assoc(t, K, V, K, V).
max_assoc(t(K,V,_,_,R), _, _, Key, Val) :-
    max_assoc(R, K, V, Key, Val).
```

Megkeresi a legnagyobb kulcsot a fában, és a hozzá tartozó értéket.

```prolog
%!  min_assoc(+Assoc, -Key, -Value) is semidet.
%
%   True if Key-Value is in assoc and Key is the smallest key.

min_assoc(t(K,V,_,L,_), Key, Val) :-
    min_assoc(L, K, V, Key, Val).

min_assoc(t, K, V, K, V).
min_assoc(t(K,V,_,L,_), _, _, Key, Val) :-
    min_assoc(L, K, V, Key, Val).
```

Ugyanez a legkisebb kulcsra.

```prolog
%!  put_assoc(+Key, +Assoc0, +Value, -Assoc) is det.
%
%   Assoc is Assoc0, except that Key is associated with
%   Value. This can be used to insert and change associations.

put_assoc(Key, A0, Value, A) :-
    insert(A0, Key, Value, A, _).

insert(t, Key, Val, t(Key,Val,-,t,t), yes).
insert(t(Key,Val,B,L,R), K, V, NewTree, WhatHasChanged) :-
    compare(Rel, K, Key),
    insert(Rel, t(Key,Val,B,L,R), K, V, NewTree, WhatHasChanged).

insert(=, t(Key,_,B,L,R), _, V, t(Key,V,B,L,R), no).
insert(<, t(Key,Val,B,L,R), K, V, NewTree, WhatHasChanged) :-
    insert(L, K, V, NewL, LeftHasChanged),
    adjust(LeftHasChanged, t(Key,Val,B,NewL,R), left, NewTree, WhatHasChanged).
insert(>, t(Key,Val,B,L,R), K, V, NewTree, WhatHasChanged) :-
    insert(R, K, V, NewR, RightHasChanged),
    adjust(RightHasChanged, t(Key,Val,B,L,NewR), right, NewTree, WhatHasChanged).
```

A beszúrásnál először az `insert/5` szabály hívódik meg, aminek utolsó argumentuma azt mondja meg, hogy nőtt-e a fa mélysége. Ez az üres fa esetét lekezeli, egyébként pedig a felelősséget az `insert/6` szabályra hárítja. Ennek az argumentumai:

1. A gyökérben levő kulcs hogyan viszonyul a beszúrandó kulcshoz (`<`, `=`, `>`).

2. Az eredeti AVL-fa.

3. A beszúrandó kulcs.

4. A beszúrandó érték.

5. Az új AVL-fa.

6. Nőtt-e a fa mélysége.

Ha a kulcsok megegyeznek, akkor a hozzá tartozó értéket a megadottra lecseréli. Ha a beszúrandó kulcs a kisebb, akkor a baloldali részfán végez rekurzívan beszúrást (az `insert/5`-tel), majd az `adjust` szabállyal (ld. lent) biztosítja a kiegyensúlyozottságot; ha a beszúrandó kulcs a nagyobb, akkor ugyanez a jobboldali részfával.

```prolog
adjust(no, Oldree, _, Oldree, no).
adjust(yes, t(Key,Val,B0,L,R), LoR, NewTree, WhatHasChanged) :-
    table(B0, LoR, B1, WhatHasChanged, ToBeRebalanced),
    rebalance(ToBeRebalanced, t(Key,Val,B0,L,R), B1, NewTree, _, _).

%     balance  where     balance  whole tree  to be
%     before   inserted  after    increased   rebalanced
table(-      , left    , <      , yes       , no    ) :- !.
table(-      , right   , >      , yes       , no    ) :- !.
table(<      , left    , -      , no        , yes   ) :- !.
table(<      , right   , -      , no        , no    ) :- !.
table(>      , left    , -      , no        , no    ) :- !.
table(>      , right   , -      , no        , yes   ) :- !.
```

Az `adjust` első argumentuma, hogy történt-e beszúrás. Ha nem, akkor a kulcsok változatlanok, tehát a kiegyensúlyozottság továbbra is teljesül. A második argumentum a módosított AVL-fa, a harmadik azt mondja meg, hogy a bal vagy jobb részfát módosítottuk (`left` ill. `right`), a negyedik a kiegyensúlyozás után kapott AVL-fa, az utolsó pedig azt jelzi, hogy nőtt-e a fa mélysége.

A `table` táblázatból könnyen kiolvasható, hogy a 6 lehetséges esetben mi történik: mi lesz az új egyensúly-szimbólum (`<`, `-` vagy `>`), megnövekedik a mélység, illetve szükség van-e forgatásra. Látszik, hogy csak két esetben van szükség forgatásra. Nézzük meg, mi történik az alábbi AVL-fával a 4-es kulcs beszúrásakor!

```
       8                   8                   6
      / \                 / \                 / \
     /   \               /   \               /   \
    /     \             /     \             /     \
   3      10  --->     3      10  --->     3       8
  / \                 / \                 / \       \
 /   \               /   \               /   \       \
1     6             1     6             1     4      10
                         /
                        4
```

A 4-es beszúrása után a 6-os `<` típusú lesz, a 3-as `>` típusú, de probléma csak a legfelső szinten jelentkezik, ahol a 8-as már eleve `<` típusú volt. Itt tehát az egész fára fog meghívódni a forgató `rebalance` operáció, aminek az eredménye jobboldalt látszik.

```prolog
%!  del_min_assoc(+Assoc0, ?Key, ?Val, -Assoc) is semidet.
%
%   True if Key-Value  is  in  Assoc0   and  Key  is  the smallest key.
%   Assoc is Assoc0 with Key-Value   removed. Warning: This will
%   succeed with _no_ bindings for Key or Val if Assoc0 is empty.

del_min_assoc(Tree, Key, Val, NewTree) :-
    del_min_assoc(Tree, Key, Val, NewTree, _DepthChanged).

del_min_assoc(t(Key,Val,_B,t,R), Key, Val, R, yes) :- !.
del_min_assoc(t(K,V,B,L,R), Key, Val, NewTree, Changed) :-
    del_min_assoc(L, Key, Val, NewL, LeftChanged),
    deladjust(LeftChanged, t(K,V,B,NewL,R), left, NewTree, Changed).
```

Kitörli a legkisebb kulcsú elemet. `Val` a hozzá tartozó érték, és az utolsó argumentum a törléssel keletkezett AVL-fa. Ha a baloldali részfa üres, akkor a keresett kulcs a gyökérben van, és az új fa a jobboldali részfa lesz. Egyébként a baloldali részfában végezzük rekurzívan a törlést, és utána kiegyensúlyozzuk a `deladjust` szabály segítségével (ld. lent).

```prolog
%!  del_max_assoc(+Assoc0, ?Key, ?Val, -Assoc) is semidet.
%
%   True if Key-Value  is  in  Assoc0   and  Key  is  the greatest key.
%   Assoc is Assoc0 with Key-Value   removed. Warning: This will
%   succeed with _no_ bindings for Key or Val if Assoc0 is empty.

del_max_assoc(Tree, Key, Val, NewTree) :-
    del_max_assoc(Tree, Key, Val, NewTree, _DepthChanged).

del_max_assoc(t(Key,Val,_B,L,t), Key, Val, L, yes) :- !.
del_max_assoc(t(K,V,B,L,R), Key, Val, NewTree, Changed) :-
    del_max_assoc(R, Key, Val, NewR, RightChanged),
    deladjust(RightChanged, t(K,V,B,L,NewR), right, NewTree, Changed).
```

Ugyanez, csak a legnagyobb kulcsú elemmel és jobboldali rekurzióval.

```prolog
%!  del_assoc(+Key, +Assoc0, ?Value, -Assoc) is semidet.
%
%   True if Key-Value is  in  Assoc0.   Assoc  is  Assoc0 with
%   Key-Value removed.

del_assoc(Key, A0, Value, A) :-
    delete(A0, Key, Value, A, _).

% delete(+Subtree, +SearchedKey, ?SearchedValue, ?SubtreeOut, ?WhatHasChanged)
delete(t(Key,Val,B,L,R), K, V, NewTree, WhatHasChanged) :-
    compare(Rel, K, Key),
    delete(Rel, t(Key,Val,B,L,R), K, V, NewTree, WhatHasChanged).

% delete(+KeySide, +Subtree, +SearchedKey, ?SearchedValue, ?SubtreeOut, ?WhatHasChanged)
% KeySide is an operator {<,=,>} indicating which branch should be searched for the key.
% WhatHasChanged {yes,no} indicates whether the NewTree has changed in depth.
delete(=, t(Key,Val,_B,t,R), Key, Val, R, yes) :- !.
delete(=, t(Key,Val,_B,L,t), Key, Val, L, yes) :- !.
delete(=, t(Key,Val,>,L,R), Key, Val, NewTree, WhatHasChanged) :-
    % Rh tree is deeper, so rotate from R to L
    del_min_assoc(R, K, V, NewR, RightHasChanged),
    deladjust(RightHasChanged, t(K,V,>,L,NewR), right, NewTree, WhatHasChanged),
    !.
delete(=, t(Key,Val,B,L,R), Key, Val, NewTree, WhatHasChanged) :-
    % Rh tree is not deeper, so rotate from L to R
    del_max_assoc(L, K, V, NewL, LeftHasChanged),
    deladjust(LeftHasChanged, t(K,V,B,NewL,R), left, NewTree, WhatHasChanged),
    !.

delete(<, t(Key,Val,B,L,R), K, V, NewTree, WhatHasChanged) :-
    delete(L, K, V, NewL, LeftHasChanged),
    deladjust(LeftHasChanged, t(Key,Val,B,NewL,R), left, NewTree, WhatHasChanged).
delete(>, t(Key,Val,B,L,R), K, V, NewTree, WhatHasChanged) :-
    delete(R, K, V, NewR, RightHasChanged),
    deladjust(RightHasChanged, t(Key,Val,B,L,NewR), right, NewTree, WhatHasChanged).
```

Általános törlő operáció. Nézzük végig a `delete/6` egyes eseteit!

1. Keresett kulcs a gyökérben, baloldali részfa üres. Eredmény a jobboldali részfa.

2. Keresett kulcs a gyökérben, jobboldali részfa üres. Eredmény a baloldali részfa.

3. Keresett kulcs a gyökérben, és a jobboldali részfa mélyebb. Ekkor a jobboldali részfából kiveszi a legkisebb kulcsú elemet, és berakja a gyökérbe, majd kiegyensúlyozza a fát.

4. Keresett kulcs a gyökérben, és a jobboldali részfa nem mélyebb. Ekkor a baloldali részfából kiveszi a legnagyobb kulcsú elemet, és berakja a gyökérbe, majd kiegyensúlyozza a fát.

5. Keresett kulcs kisebb a gyökér kulcsánál. Rekurzív törlés a bal részfában, utána kiegyensúlyozás.

6. Keresett kulcs nagyobb a gyökér kulcsánál. Rekurzív törlés a jobb részfában, utána kiegyensúlyozás.

```prolog
deladjust(no, OldTree, _, OldTree, no).
deladjust(yes, t(Key,Val,B0,L,R), LoR, NewTree, RealChange) :-
    deltable(B0, LoR, B1, WhatHasChanged, ToBeRebalanced),
    rebalance(ToBeRebalanced, t(Key,Val,B0,L,R), B1, NewTree, WhatHasChanged, RealChange).

%     balance  where     balance  whole tree  to be
%     before   deleted   after    changed   rebalanced
deltable(-      , right   , <      , no        , no    ) :- !.
deltable(-      , left    , >      , no        , no    ) :- !.
deltable(<      , right   , -      , yes       , yes   ) :- !.
deltable(<      , left    , -      , yes       , no    ) :- !.
deltable(>      , right   , -      , yes       , no    ) :- !.
deltable(>      , left    , -      , yes       , yes   ) :- !.
% It depends on the tree pattern in avl_geq whether it really decreases.
```

Teljesen hasonló a beszúrás utáni `adjust` szabályhoz, csak törlés után.

```prolog
% Single and double tree rotations - these are common for insert and delete.
/* The patterns (>)-(>), (>)-( <), ( <)-( <) and ( <)-(>) on the LHS
   always change the tree height and these are the only patterns which can
   happen after an insertion. That's the reason why we can use a table only to
   decide the needed changes.

   The patterns (>)-( -) and ( <)-( -) do not change the tree height. After a
   deletion any pattern can occur and so we return yes or no as a flag of a
   height change.  */


rebalance(no, t(K,V,_,L,R), B, t(K,V,B,L,R), Changed, Changed).
rebalance(yes, OldTree, _, NewTree, _, RealChange) :-
    avl_geq(OldTree, NewTree, RealChange).
```

Elérkeztünk az AVL-fák lelkéhez, a forgató `rebalance` szabályhoz. Az első argumentum azt mondja meg, hogy szükség van-e forgatásra. Ha ez `no`, akkor a fa lényegileg nem változik, csak az egyensúly-szimbólumot állítja be a harmadik argumentumban kapott értékre (ami a megfelelő táblázatból lett kiolvasva). Az utolsó argumentum azt mutatja, hogy a fa mélysége megváltozott-e, és ha nem történik forgatás, akkor csak átmásolja a beszúrás/törlés során megállapított értéket.

A tényleges forgatást az `avl_geq` végzi, aminek mindössze 3 argumentuma van: a régi fa, a forgatás után keletkező új fa, és hogy a forgatás során változott-e a fa mélysége.

```prolog
avl_geq(t(A,VA,>,Alpha,t(B,VB,>,Beta,Gamma)),
        t(B,VB,-,t(A,VA,-,Alpha,Beta),Gamma), yes) :- !.
avl_geq(t(A,VA,>,Alpha,t(B,VB,-,Beta,Gamma)),
        t(B,VB,<,t(A,VA,>,Alpha,Beta),Gamma), no) :- !.
avl_geq(t(B,VB,<,t(A,VA,<,Alpha,Beta),Gamma),
        t(A,VA,-,Alpha,t(B,VB,-,Beta,Gamma)), yes) :- !.
avl_geq(t(B,VB,<,t(A,VA,-,Alpha,Beta),Gamma),
        t(A,VA,>,Alpha,t(B,VB,<,Beta,Gamma)), no) :- !.
avl_geq(t(A,VA,>,Alpha,t(B,VB,<,t(X,VX,B1,Beta,Gamma),Delta)),
        t(X,VX,-,t(A,VA,B2,Alpha,Beta),t(B,VB,B3,Gamma,Delta)), yes) :-
    !,
    table2(B1, B2, B3).
avl_geq(t(B,VB,<,t(A,VA,>,Alpha,t(X,VX,B1,Beta,Gamma)),Delta),
        t(X,VX,-,t(A,VA,B2,Alpha,Beta),t(B,VB,B3,Gamma,Delta)), yes) :-
    !,
    table2(B1, B2, B3).

table2(< ,- ,> ).
table2(> ,< ,- ).
table2(- ,- ,- ).
```

Nézzük végig az egyes eseteket!

1. `>/>`: a jobboldali részfa túl mély, és annak a jobboldali részfája a mélyebb. A forgatás hatására a mélység csökken.

           A                  B
          / \                / \
         /   \              /   \
        α     B    --->    A     γ
             / \          / \
            β   γ        α   β

2. `>/-`: a jobboldali részfa túl mély, és az egyensúlyban van. Ez csak baloldali törlés után jöhet létre. A forgatás megegyezik az előzővel, de ilyenkor a mélység nem változik.

3. `</<`: az 1-es tükrözve.

4. `</-`: a 2-es tükrözve.

5. `>/<`: a jobboldali részfa túl mély, és annak a baloldali részfája a mélyebb. A forgatás hatására a mélység csökken. Az `A`-nál és `B`-nél levő egyensúly-szimbólumokat az `X`-nél levő alapján a `table2` táblázat adja meg.

            A                      X
           / \                    / \
          /   \                  /   \
         /     \                /     \
        α       B     --->     A       B
               / \            / \     / \
              /   \          /   \   /   \
             X     δ        α     β γ     δ
            / \
           β   γ

6. `</>`: az 5-ös tükrözve (ilyen volt a fenti beszúrásos példa).

Itt érdemes megjegyezni, hogy a mélységcsökkenést jelző utolsó argumentumot csak a `deladjust` veszi figyelembe, az `adjust` nem. Ennek az az oka, hogy ha beszúráskor nőne a mélység, akkor a forgatás azt mindig kompenzálja, tehát végül az eredeti állapothoz képest a mélység nem változik. Ezzel szemben törlésnél ha forgatásra van szükség, akkor csak ettől függ, hogy a mélység csökken-e.

```prolog
                 /*******************************
                 *            ERRORS            *
                 *******************************/

:- multifile
    error:has_type/2.

error:has_type(assoc, X) :-
    (   X == t
    ->  true
    ;   compound(X),
        functor(X, t, 5)
    ).
```

A `:- multifile` figyelmezteti a fordítót, hogy az utána következő szabály részei több fájlban találhatóak. Az `error:` azt mondja meg, hogy bár most az `assoc` modulban vagyunk, a `has_type` szabály az `error` modulhoz tartozik.

Maga a `has_type` szabály, ahogy arról röviden már szó volt, azt ellenőrzi, hogy `X` egy AVL-fa-e. Mivel ez gyakran meghívódik, itt nem történik olyan részletes ellenőrzés, mint az `is_assoc` esetén. Az `X` megfelel, ha üres fa (a `t` atom), vagy ha egy 5-elemű struktúra, aminek a feje `t`.

Ezzel vége a könyvtár forráskódjának. Ez szerintem egy szép, jól érthető program, ami kevés külső definíciót használ, ugyanakkor rendkívül tanulságos.

## Feladat

Egy másik gyakran szükséges adatszerkezet a *prioritásos sor*, amiben minden elemhez egy prioritást rendelünk, és mindig a legmagasabb prioritásút vesszük ki belőle. A következő szabályokkal kezelhető:

- `üres_sor(?Sor)` [semidet]

- `sorba_tesz(+Sor, +Prioritás, ?Elem, -Sor1)` [det]

- `maximum_elem(+Sor, ?Elem)` [semidet]

- `maximumot_kivesz(+Sor, -Prioritás, ?Elem, -Sor1)` [semidet]

A kényelmes használat kedvéért még érdemes listából/listává átváltó szabáyokat is készíteni (ahol a lista `Prioritás-Elem` párokból áll):

- `listából_sor(+Lista, -Sor)` [det]

- `sorból_lista(+Sor, -Lista)` [det]

Ennek egy egyszerű megoldása az, hogy a betevés sorrendjében egy listában tároljuk az elemeket - ebben az esetben a maximum megkeresése ill. a sorból kivétel *O*(*n*) műveletigényű lesz. Egy másik lehetőség, hogy az elemeket mindig csökkenő sorrendben tároljuk; ekkor az új elem betevése lesz *O*(*n*)-es komplexitású.

Egy hatékonyabb módszert kapunk, ha itt is egy fát használunk. Ebben a fában a csúcsokból nem csak kettő, hanem több ág is indulhat, és csak azt várjuk el, hogy a csúcsban levő prioritás legalább akkora legyen, mint az alatta levő részfák maximum prioritása, így a maximum mindig a gyökérben lesz. Ezt "párosító kupacnak" (*pairing heap*) nevezik.

Két ilyen fa egybeolvasztása (*meld*) nagyon egyszerű: megnézzük, hogy melyik gyökerének nagyobb a prioritása, és az marad a gyökér, a másik pedig ez alá kerül új részfaként. A beszúrás is ennek egy speciális esete, ahol a beszúrt elem egy olyan fa, aminek csak gyökere van.

Az egyetlen bonyolultabb - *O*(log *n*) komplexitású - művelet a maximális prioritású elem kivétele. Ilyenkor az összes alatta levő részfát egybe kell olvasztani, amíg csak egy nem marad. Ez sokféleképp megtehető, és a különböző módszerek más jellegű fákat eredményeznek. A klasszikus megoldás az, hogy a részfákat először balról jobbra páronként egybeolvasztjuk (innen a nevében a *párosító*), és utána a jobboldalitól elindulva a már egybeolvasztott párokat egyenként hozzáolvasztjuk. (Ez rekurzív módon nagyon egyszerűen megfogalmazható.)

Nézzünk egy példát! Jelölje `P-[P1,P2,..,Pn]` azt a fát, aminek a gyökerében `P`, az alatta levő részfák gyökerében pedig `Pi` prioritások vannak (a további, érdektelen részfákat `..`-al jelöltem):

```
10-[5,8,2,8,10,1,3]

(maximumkivétel => 7 különálló fa)

5-[..] 8-[..] 2-[..] 8-[..] 10-[..] 1-[..] 3-[..]

(párosítás balról jobbra => 4 különálló fa)

8-[5,..] 8-[2,..] 10-[1,..] 3-[..]

(egyesítés jobbról balra, amíg csak 1 marad)

8-[5,..] 8-[2,..] 10-[3,1,..]

8-[5,..] 10-[8-[2,..],3,1,..]

10-[8-[5,..],8-[2,..],3,1,..]
```

Készítsetek egy prioritásos sor adatszerkezetet párosító kupaccal, és aztán hasonlítsátok össze a megoldást az [SWI Prologban levővel](https://github.com/SWI-Prolog/swipl-devel/blob/master/library/heaps.pl)!

## Megjegyzések

Egy másik, kicsit bonyolultabb keresőfa a *piros-fekete fa*, ami az egyes elemekhez (piros vagy fekete) színt rendel, és ennek segítségével még hatékonyabb beszúrást/törlést tesz lehetővé, mint az AVL-fa. Cserébe viszont a keresés egy kicsit lassabb lehet. (Az SWI Prolog piros-fekete fákat használ *tömb*-jellegű adatstruktúra megvalósítására.)

A kulcs szerinti keresés problémájára még egy nagyon érdekes megoldást adnak a *hash táblák*, melyeknek rengeteg variánsa létezik. Ezek általában sokkal gyorsabbak, mint a keresőfák (*O*(1) átlagosan), de időnként lassabbak is lehetnek (*O*(*n*) legrosszabb esetben), valamint az elemeket nem rendezetten tárolják.

Kereséssel kapcsolatos témákról a klasszikus hivatkozás:

D.E. Knuth, *The Art of Computer Programming, Vol. 3 - Sorting and Searching*, 2nd Ed., Addison-Wesley, 1998.

Az első kiadás megjelent magyarul is:

D.E. Knuth, *A számítógép-programozás művészete 3 - Keresés és rendezés*, Műszaki Könyvkiadó, 1988.

Egy modern, átfogó könyv algoritmusokról és adatszerkezetekről:

R. Sedgewick, K. Wayne, *Algorithms*, 4th Ed., Addison-Wesley, 2011.

Illetve egy rövid, képekkel teli, olvasmányos könyv ugyanerről:

A.Y. Bhargava, *Grokking Algorithms*, Manning, 2016.

A fentiek mind általános referenciák, és a bennük szereplő adatszerkezetek (pl. hash táblák) nem mind alkalmazhatóak közvetlenül Prologban, ahol nincs mód egy adat *megváltoztatására*, csak egy módosított változat készítésére (tehát az adatok *perzisztensek*). Ilyen megkötések mellett a hatékonysághoz időnként trükkök kellenek - erről szól az alábbi könyv:

Ch. Okasaki, *Purely Functional Data Structures*, Cambridge, 1996.
