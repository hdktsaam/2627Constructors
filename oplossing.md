# Oplossingen: JavaScript Constructors, Private Variabelen & Getters/Setters (Constructor Functions)

Hieronder vind je de uitgewerkte oplossingen voor de 4 oefeningen, volledig geschreven met klassieke **Constructor Functions**. Private variabelen zijn afgeschermd met behulp van **closures** (lokale variabelen met `let` binnen de constructor scope), en getters/setters zijn gedefinieerd met `Object.defineProperty` of `Object.defineProperties`.

---

## Oefening 1: Bankrekening (`BankRekening`)

```javascript
function BankRekening(rekeninghouder, initieelSaldo) {
  this.rekeninghouder = rekeninghouder;

  // Private variabele via closure
  let _saldo = initieelSaldo > 0 ? initieelSaldo : 0;

  // Getter en setter toevoegen
  Object.defineProperty(this, 'saldo', {
    get: function() {
      return _saldo;
    },
    set: function(nieuwSaldo) {
      if (nieuwSaldo < 0) {
        console.error("Fout: Saldo kan niet negatief zijn.");
      } else {
        _saldo = nieuwSaldo;
      }
    },
    enumerable: true,
    configurable: true
  });

  // Methoden op het object
  this.storten = function(bedrag) {
    if (bedrag <= 0) {
      console.error("Het te storten bedrag moet groter zijn dan 0.");
      return;
    }
    _saldo += bedrag;
    console.log(`€${bedrag} gestort. Nieuw saldo: €${_saldo}`);
  };

  this.opnemen = function(bedrag) {
    if (bedrag > _saldo) {
      console.error("Onvoldoende saldo op de rekening.");
    } else if (bedrag <= 0) {
      console.error("Het op te nemen bedrag moet groter zijn dan 0.");
    } else {
      _saldo -= bedrag;
      console.log(`€${bedrag} opgenomen. Resterend saldo: €${_saldo}`);
    }
  };
}

// Voorbeeld van gebruik:
const mijnRekening = new BankRekening("Jan Jansen", 100);
console.log(mijnRekening.saldo); // 100
console.log(mijnRekening._saldo); // undefined (private!)

mijnRekening.saldo = -50;        // Fout: Saldo kan niet negatief zijn.
mijnRekening.storten(50);        // €50 gestort. Nieuw saldo: €150
mijnRekening.opnemen(200);       // Onvoldoende saldo op de rekening.
```

---

## Oefening 2: Thermostaat (`Thermostaat`)

```javascript
function Thermostaat(temperatuurCelsius) {
  // Private variabele
  let _temperatuurCelsius;

  Object.defineProperties(this, {
    // Getter en setter voor Celsius
    temperatuurCelsius: {
      get: function() {
        return _temperatuurCelsius;
      },
      set: function(waarde) {
        if (waarde < -50 || waarde > 100) {
          console.error("Temperatuur buiten het toegestane bereik (-50°C tot 100°C).");
        } else {
          _temperatuurCelsius = waarde;
        }
      },
      enumerable: true,
      configurable: true
    },

    // Getter en setter voor Fahrenheit
    temperatuurFahrenheit: {
      get: function() {
        return _temperatuurCelsius * 1.8 + 32;
      },
      set: function(waardeInFahrenheit) {
        const celsius = (waardeInFahrenheit - 32) / 1.8;
        this.temperatuurCelsius = celsius; // Gebruikt de setter van Celsius voor validatie
      },
      enumerable: true,
      configurable: true
    }
  });

  // Initialisatie via de setter (voor de bereik-controle)
  this.temperatuurCelsius = temperatuurCelsius;
}

// Voorbeeld van gebruik:
const t = new Thermostaat(20);
console.log(`${t.temperatuurCelsius}°C`);    // 20°C
console.log(`${t.temperatuurFahrenheit}°F`); // 68°F

t.temperatuurFahrenheit = 86;
console.log(`${t.temperatuurCelsius}°C`);    // 30°C

t.temperatuurCelsius = 150;                  // Foutmelding buiten bereik
```

---

## Oefening 3: Werknemer & Salaris (`Werknemer`)

```javascript
function Werknemer(naam, functie, maandsalaris) {
  this.naam = naam;
  this.functie = functie;

  // Private variabele in de constructor scope
  let _maandsalaris = maandsalaris > 0 ? maandsalaris : 0;

  Object.defineProperties(this, {
    // Getter & Setter voor maandsalaris
    maandsalaris: {
      get: function() {
        return _maandsalaris;
      },
      set: function(nieuwSalaris) {
        if (nieuwSalaris < _maandsalaris) {
          console.error("Salarisverlaging is niet toegestaan.");
        } else {
          _maandsalaris = nieuwSalaris;
        }
      },
      enumerable: true,
      configurable: true
    },

    // Getter-only voor jaarsalaris (inclusief 8% vakantiegeld)
    jaarsalaris: {
      get: function() {
        const vakantiegeld = _maandsalaris * 12 * 0.08;
        return (_maandsalaris * 12) + vakantiegeld;
      },
      enumerable: true,
      configurable: true
    }
  });
}

// Voorbeeld van gebruik:
const w = new Werknemer("Sophie", "Developer", 3000);
console.log(`Maandsalaris: €${w.maandsalaris}`); // €3000
console.log(`Jaarsalaris: €${w.jaarsalaris}`);   // €38880

w.maandsalaris = 2800; // Fout: Salarisverlaging is niet toegestaan.
w.maandsalaris = 3500; // Salaris verhoogd!
console.log(`Nieuw jaarsalaris: €${w.jaarsalaris}`); // €45360
```

---

## Oefening 4: Gebruikersprofiel met Wachtwoord (`Gebruiker`)

```javascript
function Gebruiker(gebruikersnaam, initieelWachtwoord) {
  this.gebruikersnaam = gebruikersnaam;

  // Private variabele
  let _wachtwoord;

  Object.defineProperty(this, 'wachtwoord', {
    // Getter geeft gemaskerd wachtwoord terug
    get: function() {
      return "******";
    },
    // Setter controleert lengte (min 8) en minstens 1 cijfer
    set: function(nieuwWachtwoord) {
      const heeftMinAchtTekens = nieuwWachtwoord.length >= 8;
      const heeftCijfer = /\d/.test(nieuwWachtwoord);

      if (!heeftMinAchtTekens || !heeftCijfer) {
        console.error("Wachtwoord ongeldig. Minstens 8 tekens en minimaal 1 cijfer vereist.");
      } else {
        _wachtwoord = nieuwWachtwoord;
        console.log("Wachtwoord succesvol opgeslagen.");
      }
    },
    enumerable: true,
    configurable: true
  });

  // Methode om private wachtwoord te valideren
  this.valideerWachtwoord = function(invoer) {
    return _wachtwoord === invoer;
  };

  // Initialisatie van wachtwoord via de setter
  this.wachtwoord = initieelWachtwoord;
}

// Voorbeeld van gebruik:
const gebruiker = new Gebruiker("Karel123", "Geheim123");

console.log(gebruiker.wachtwoord); // "******" (gemaskerd)
console.log(gebruiker._wachtwoord); // undefined

gebruiker.wachtwoord = "kort";     // Foutmelding: te kort en geen cijfer
gebruiker.wachtwoord = "veiligWachtwoord123"; // Succesvol aangepast

console.log(gebruiker.valideerWachtwoord("foutWachtwoord")); // false
console.log(gebruiker.valideerWachtwoord("veiligWachtwoord123")); // true
```
