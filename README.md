
# EDU SIS User Client (Unofficial) (file soon)

**Compatibel met:** ParnasSys (NL) — *onofficieel, niet gelieerd en niet ondersteund door de leverancier.*

Een client library voor het beheren van medewerkers/accounts via een REST-achtige integratielaag, bedoeld om beheerhandelingen te automatiseren vanuit **.NET** en **Python** applicaties.

> **Disclaimer (lees dit eerst)**
> - Dit is een **third-party / onofficieel** project en **niet** verbonden aan, goedgekeurd door, of ondersteund door de leverancier(s) van ParnasSys.
> - Gebruik uitsluitend als je **expliciete toestemming** hebt van de gegevensverantwoordelijke (school/bestuur) én als jouw gebruik past binnen de **voorwaarden/contracten** van de betreffende dienst.
> - “ParnasSys” is een handelsnaam/merk van de rechthebbende en wordt hier alleen genoemd om **compatibiliteit** te beschrijven. Alle merken zijn eigendom van hun respectieve houders.

---

##  Features

- ✅ Authenticatie met beheerdersaccount (sessie-gebaseerd)
- ✅ Zoeken naar medewerkers (op naam of e-mail)
- ✅ Aanmaken van nieuwe medewerkers
- ✅ Activeren / Deactiveren van medewerkers
- ✅ Verwijderen van medewerkers (met bevestiging)
- ✅ (Optioneel) Identity-koppeling (bijv. Azure AD) *indien beschikbaar binnen je omgeving*
- ✅ Health check / beschikbaarheidscontrole
- ✅ Uitgebreide error handling met gestructureerde responses

---

## 📦 .NET DLL Gebruik

### Vereisten
- .NET Framework 4.7.2 of hoger
- Newtonsoft.Json

### Installatie
1. Download de dll uit de **Releases**
2. Voeg een referentie toe aan je project
3. Zorg dat `Newtonsoft.Json.dll` beschikbaar is (meestal automatisch via NuGet)

### Gebruik in C#

```csharp
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        // Maak client aan
        var client = new SisClient();

        // Login met admin credentials
        var loginResult = await client.LoginAsync("admin@school.nl", "wachtwoord");

        if (!loginResult.Success)
        {
            Console.WriteLine($"Login mislukt: {loginResult.ErrorMessage}");
            Console.WriteLine($"Error code: {loginResult.ErrorCode}");
            return;
        }

        Console.WriteLine("Succesvol ingelogd!");

        // Zoek medewerkers op naam
        var searchResult = await client.SearchUsersAsync("Jan");

        if (searchResult.Success)
        {
            Console.WriteLine($"Gevonden: {searchResult.Count} medewerker(s)");
            foreach (var user in searchResult.Users)
            {
                Console.WriteLine($"  - {user.Naam} ({user.Email})");
                Console.WriteLine($"    Functie: {user.Functie}, Actief: {user.Actief}");
            }
        }
        else
        {
            Console.WriteLine($"Zoeken mislukt: {searchResult.ErrorMessage} ({searchResult.ErrorCode})");
        }

        // Maak nieuwe medewerker aan
        var createResult = await client.CreateUserAsync(
            achternaam: "Jansen",
            roepnaam: "Jan",
            email: "j.jansen@school.nl",
            gebruikersnaam: "j.jansen",
            tussenvoegsel: "van",   // Optioneel
            linkIdentity: true      // Optioneel, default: true (bijv. Azure AD)
        );

        if (createResult.Success)
        {
            Console.WriteLine("Medewerker aangemaakt!");
            Console.WriteLine($"Identity gekoppeld: {createResult.IdentityLinked}");
        }
        else
        {
            Console.WriteLine($"Fout: {createResult.ErrorMessage}");
            Console.WriteLine($"Error code: {createResult.ErrorCode}");
        }

        // Activeer / Deactiveer medewerker
        await client.ActivateUserAsync("j.jansen@school.nl");
        await client.DeactivateUserAsync("j.jansen@school.nl");

        // Verwijder medewerker (vereist naam voor bevestiging)
        var deleteResult = await client.DeleteUserAsync(
            "j.jansen@school.nl",
            "Jan van Jansen"
        );

        Console.WriteLine(deleteResult.Success ? "Medewerker verwijderd" : $"Verwijderen mislukt: {deleteResult.ErrorMessage}");

        // Health check
        var health = await client.CheckHealthAsync();
        if (health.Success)
        {
            Console.WriteLine($"Status: {health.Status}");
            Console.WriteLine($"Versie: {health.Version}");
        }

        // Logout
        await client.LogoutAsync();
        Console.WriteLine("Uitgelogd");
    }
}
````

### Gebruik in VB.NET

```vbnet
Imports System.Threading.Tasks

Module Program
    Sub Main()
        MainAsync().GetAwaiter().GetResult()
    End Sub

    Async Function MainAsync() As Task
        Dim client As New SisClient()

        Dim loginResult = Await client.LoginAsync("admin@school.nl", "wachtwoord")
        If Not loginResult.Success Then
            Console.WriteLine($"Login mislukt: {loginResult.ErrorMessage} ({loginResult.ErrorCode})")
            Return
        End If

        Console.WriteLine("Succesvol ingelogd!")

        Dim searchResult = Await client.SearchUsersAsync("Jan")
        If searchResult.Success Then
            Console.WriteLine($"Gevonden: {searchResult.Count} medewerker(s)")
            For Each user In searchResult.Users
                Console.WriteLine($"- {user.Naam} ({user.Email})")
            Next
        Else
            Console.WriteLine($"Zoeken mislukt: {searchResult.ErrorMessage} ({searchResult.ErrorCode})")
        End If

        Dim createResult = Await client.CreateUserAsync(
            achternaam:="Jansen",
            roepnaam:="Jan",
            email:="j.jansen@school.nl",
            gebruikersnaam:="j.jansen",
            tussenvoegsel:="van",      ' Optioneel
            linkIdentity:=True         ' Optioneel, default: True
        )

        If createResult.Success Then
            Console.WriteLine($"Medewerker aangemaakt! Identity: {createResult.IdentityLinked}")
        Else
            Console.WriteLine($"Fout: {createResult.ErrorMessage} ({createResult.ErrorCode})")
        End If

        Await client.LogoutAsync()
        Console.WriteLine("Uitgelogd")
    End Function
End Module
```

---

## 🐍 Python Client Gebruik

### Vereisten

* Python 3.7 of hoger
* `requests` library

### Installatie

```bash
pip install requests
```

Kopieer `sis_client.py` naar je project of voeg het toe aan je Python path.

### Gebruik

```python
from sis_client import SisClient

client = SisClient()

login = client.login("admin@school.nl", "wachtwoord")
if not login.success:
    print(f"Login mislukt: {login.error} ({login.error_code})")
    raise SystemExit(1)

print("Succesvol ingelogd!")

users = client.search_users("Jan", search_type="name")
print(f"Gevonden: {len(users)} medewerker(s)")
for u in users:
    print(f"  - {u['naam']} ({u['email']})")

create = client.create_user_result(
    achternaam="Jansen",
    roepnaam="Jan",
    email="j.jansen@school.nl",
    gebruikersnaam="j.jansen",
    tussenvoegsel="van",
    link_identity=True
)

if create.success:
    print("Medewerker aangemaakt!")
    print(f"Identity gekoppeld: {create.data.get('identity_linked', False)}")
else:
    print(f"Fout: {create.error} ({create.error_code})")

client.logout()
print("Uitgelogd")
```

---

## 📚 API Referentie

### Authenticatie

| Methode                                                  | Beschrijving                     | Parameters                          |
| -------------------------------------------------------- | -------------------------------- | ----------------------------------- |
| `LoginAsync(email, password)` / `login(email, password)` | Inloggen (start sessie)          | `email`: string, `password`: string |
| `LogoutAsync()` / `logout()`                             | Uitloggen en sessie beëindigen   | -                                   |
| `IsAuthenticated` / `is_authenticated`                   | Controleer of client is ingelogd | -                                   |

### Gebruikersbeheer

| Methode                                                                    | Beschrijving                         | Parameters                                                                 |
| -------------------------------------------------------------------------- | ------------------------------------ | -------------------------------------------------------------------------- |
| `SearchUsersAsync(query, searchType)` / `search_users(query, search_type)` | Zoek medewerkers                     | `query`: string, `searchType`: enum / `search_type`: `"name"` of `"email"` |
| `CreateUserAsync(...)` / `create_user(...)`                                | Maak nieuwe medewerker aan           | Zie *CreateUser Parameters*                                                |
| `DeleteUserAsync(email, naam)` / `delete_user(email, naam)`                | Verwijder medewerker (bevestiging)   | `email`: string, `naam`: string                                            |
| `ActivateUserAsync(email)` / `activate_user(email)`                        | Activeer medewerker                  | `email`: string                                                            |
| `DeactivateUserAsync(email)` / `deactivate_user(email)`                    | Deactiveer medewerker                | `email`: string                                                            |
| `ToggleUserAsync(email, active)` / `toggle_user(email, active)`            | Toggle status                        | `email`: string, `active`: bool                                            |
| `LinkIdentityAsync(email)` / `link_identity(email)`                        | Koppel identity provider (optioneel) | `email`: string                                                            |

### Health Check

| Methode                                 | Beschrijving              | Parameters |
| --------------------------------------- | ------------------------- | ---------- |
| `CheckHealthAsync()` / `check_health()` | Controleer service status | -          |

---

## 📋 CreateUser Parameters

| Parameter                        | Type   | Verplicht | Beschrijving                                 |
| -------------------------------- | ------ | --------- | -------------------------------------------- |
| `achternaam`                     | string | ✅         | Achternaam van de medewerker                 |
| `roepnaam`                       | string | ✅         | Voornaam/roepnaam                            |
| `email`                          | string | ✅         | E-mailadres (moet uniek zijn)                |
| `gebruikersnaam`                 | string | ✅         | Gebruikersnaam voor login                    |
| `tussenvoegsel`                  | string | ❌         | Tussenvoegsel (bijv. "van", "de", "van der") |
| `linkIdentity` / `link_identity` | bool   | ❌         | Identity-koppeling (default: `true`)         |

---

## 🔄 Response Types

### .NET Response Types

Alle async methodes retourneren specifieke result types:

* **`ApiResult`**: Basis resultaat met `Success`, `Message`, `ErrorMessage`, `ErrorCode`
* **`SearchResult`**: Erft van `ApiResult`, bevat `Users` en `Count`
* **`CreateUserResult`**: Erft van `ApiResult`, bevat `IdentityLinked` (bool)
* **`ToggleResult`**: Erft van `ApiResult`, bevat `Active` (bool)
* **`HealthResult`**: Erft van `ApiResult`, bevat `Status` en `Version`

### Python Response Types

* **Eenvoudige methodes**: Retourneren `bool` (succes/falen) of `List[Dict]` (voor search)
* **`_result` varianten**: Retourneren `ApiResult` (dataclass) met volledige response info

---

## ⚠️ Error Handling

Voorbeeld (.NET):

```csharp
var result = await client.CreateUserAsync(...);
if (result.Success)
{
    Console.WriteLine(result.Message);
}
else
{
    Console.WriteLine($"Fout: {result.ErrorMessage} ({result.ErrorCode})");
}
```

Voorbeeld (Python):

```python
result = client.create_user_result(...)
if result.success:
    print(result.message)
else:
    print(f"Fout: {result.error} ({result.error_code})")
```

---



## Trademark notice

Dit project is een onafhankelijke, onofficiële client. Alle merknamen en handelsnamen zijn eigendom van hun respectieve houders en worden uitsluitend gebruikt om compatibiliteit te beschrijven.

