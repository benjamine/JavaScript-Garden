### `setTimeout` i `setInterval`

Ponieważ JavaScript jest asynchroniczny, istnieje możliwość zaplanowania wykonania 
funkcji korzystając z funkcji `setTimeout` i `setInterval`.
Since JavaScript is asynchronous, it is possible to schedule the execution of a 
function by using the `setTimeout` and `setInterval` functions.

> **Note:** Funkcje czasowe nie są częścią standardu ECMAScript. Jest to część 
> standardu [DOM][1].

    function foo() {}
    var id = setTimeout(foo, 1000); // zwraca licznę typu Number > 0

Powyższe wywołanie `setTimeout` zwraca ID budzika i planuje wywołanie `foo` za 
**około** tysiąc milisekund. `foo` zostanie wykonana dokładnie **jeden raz**.

**Nie ma pewności**, że kod zaplanowany do wykonania wykona się dokładnie po 
upłynięciu zadanego czasu podanego jako parametr do `setTimeout`, ponieważ zależy 
to od dokładności zegara w silniku JavaScript, który wykonuje kod oraz od tego, 
że inny kawałek kodu może zablokować wątek, ponieważ JavaScript jest tylko 
jedno wątkowy.

Funkcja, która została przekazana jako pierwszy parametr zostanie wykonana w 
globalnym zasięgu, co oznacza, że [`this`](#function.this) wewnątrz tej funkcji 
będzie wkazywać na obiekt *global*.

    function Foo() {
        this.value = 42;
        this.method = function() {
            // this wskazuje na obiekt global
            console.log(this.value); // wypisze undefined
        };
        setTimeout(this.method, 500);
    }
    new Foo();

> **Uwaga:** Ponieważ `setTimeout` przyjmuje **obiekt funkcji** jako pierwszy 
> argument często popełnianym błędem jest wykorzystanie składni `setTimeout(foo(), 1000)`, 
> która użyje wartości zwróconej przez funkcję `foo` jako parametru zamiast 
> funkcji `foo` samej w sobie. W większości przypadków będzie to cichy błąd, 
> ponieważ jeżeli funkcja zwróci `undefined` `setTimeout` **nie** wyrzuci żadnego 
> błędu.

### Kolejkowanie wywołań z `setInterval`

Podczas, gdy `setTimeout` wywołuje podaną funkcję tylko raz, `setInterval` - 
jak wskazuje nazwa - będzie wykonywać funkcję **w odstępach czasowych** co `X` 
milisekund. Jednakże korzystanie z tej funkcji jest odradzane.

Kiedy wykonywany kod zablokuje możliwość uruchomienia zaplanowanej funkcji, 
`setInterval` będzie próbować uruchamiać daną funkcję co będzie powodować 
kolejkowanie wykonania tej samej funkcji kilkakrotnie. W szczególności może się 
to wydarzyć przy krótkim interwale.

    function foo(){
        // coś co blokuje wykonanie na 1 sekundę 
    }
    setInterval(foo, 100);

W powyższym kodzie kod `foo` zostanie wywołany tylko raz i zablokuje wywołanie na 
jedną sekundę.

Podczas, gdy funkcja `foo` blokuje wykonanie `setInterval` będzie planować kolejne 
wywołania `foo`. W momencie, gdy pierwsze wywołanie `foo` się zakończy, już 
w kolejce do wywołania będą czekały kolejne **dziesięć** wywołań tej funkcji.

### Radzenie sobie z możliwymi blokadami

Najprostrzym jak również najbardziej kontrolowaną sytuacją jest użycie `setTimeout` 
wewnątrz wywoływanej funkcji.

    function foo(){
        // coś co blokuje wykonanie na 1 sekundę
        setTimeout(foo, 100);
    }
    foo();

Powyższy kod nie tylko hermetyzuje wywołanie `setTimeout` ale również zapobiega 
kolejkowaniu wywołań fukcji i daje dodatkową kontrolę. W tym przypadku funkcja 
`foo` może zdecydować czy powinna się wywołać ponownie czy też nie.

### Ręczne usuwanie budzików

Usuwanie budzików i interwałów dokonywane jest przez przekazanie odpowiedniego ID 
do `clearTimeout` lub `clearInterval`, w zależności z jakiej funkcji zostało 
zwrócone ID.

    var id = setTimeout(foo, 1000);
    clearTimeout(id);

### Usuwanie wszystkich budzików

Ponieważ nie istnieje wbudowana metoda na usunięcie wszystkich budzików i/lub 
interwałów, konieczne jest użycie metody brute force aby osiągnąć ten efekt.

    // usunięcie "wszystkich" budzików 
    for(var i = 1; i < 1000; i++) {
        clearTimeout(i);
    }

Nadal może istnieć jakieś budziki, na które powyższy kawałek kodu nie zadziała, 
ponieważ ID było z innego przedziału, dlatego zamiast korzystania z metody brute 
force, zaleca się śledzić wszystkie numery ID budzików, aby można je było usunąć.

### Ukryte wykorzystanie `eval`

Do `setTimeout` i `setInterval` można również przekazać string jako pierwszy 
parametr zamiast obiektu funkcji, jednakże **nigdy** nie należy korzystać z tej 
możliwości, ponieważ wewnętrznie `setTimeout` i `setInterval` wykorzystują `eval`.

> **Uwaga:** Ponieważ funkcje budzików **nie** są częścią specyfikacji standardu
> ECMAScript, działanie tych funkcji nie jest określone w momencie, gdy zostanie 
> do nich przekazany string. Na przykład Microsoftowy JScript wykorzystuje 
> konstruktor `Function` zamiast funkcji `eval`.

    function foo() {
        // zostanie wykonane 
    }

    function bar() {
        function foo() {
            // nigdy nie zostanie wywołane
        }
        setTimeout('foo()', 1000);
    }
    bar();

Ponieważ `eval` nie zostało wywołane [wprost](#core.eval) w tym przypadku, to 
string przekazany do `setTimeout` zostanie uruchomiony w *zasięgu globalnym*, 
co za tym idzie lokalna zmienna `foo` z zasięgu `bar` nie zostanie użyta.

Kolejnym zaleceniem jest aby **nie** stosować stringów do przekazywania argumentów 
do funkcji, która ma zostać wywołana przez budzik.

    function foo(a, b, c) {}
    
    // NIGDY nie należy tak robić 
    setTimeout('foo(1,2, 3)', 1000)

    // Zamiast tego należy skorzystać z anonimowej funkcji
    setTimeout(function() {
        foo(a, b, c);
    }, 1000)

>**Uwaga:** Mimo, że możliwe jest wykorzystanie składni
> `setTimeout(foo, 1000, a, b, c)`, to nie zaleca się korzystania z niej, ponieważ 
> może to prowadzić do subtelnych błędów podczas wykorzystania [metod](#function.this).

### Wnioski

Nie należy **nigdy** przekazywać stringu jako parametru do `setTimeout` lub 
`setInterval`. Jest to wyraźną oznaką **bardzo** złego kodu, jeżeli potrzebne jest 
przekazanie argumentów do funkcji należy skorzystać z *anonimowej funkcji* i 
wewnątrz niej dokonać przekazania argumentów.

Ponadto, należy unikać korzystanie z `setInterval`, ponieważ planista może 
zablokować wykonanie JavaScriptu.

[1]: http://pl.wikipedia.org/wiki/Obiektowy_model_dokumentu "Document Object Model"

