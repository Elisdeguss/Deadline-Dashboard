# Deadline Dashboard

> Projectvoorstel voor Web 3 en Cloudontwikkeling (HoGent) · Elisabeth de Gusseme · Deadline Dashboard is de werktitel van de app

## Samenvatting

Deadline Dashboard is een persoonlijk dashboard voor iedereen die moeite heeft om overzicht te houden over alle deadlines en prioriteiten te stellen. Elke taak of elk project is een bubbel. Door een sterke visualisatie zie je meteen wat dringend en belangrijk is. Om prioriteiten in te schatten gebruikt de app een veld gebaseerd op een Eisenhower-matrix.

Het project dekt twee vakken tegelijk: de C#-backend (.NET 10, Azure) voor Cloudontwikkeling en de React- en TypeScript-frontend voor Web 3.

## Het bubbelconcept

- **Kleur** = categorie (werk of persoonlijk). De echte kleuren zijn nog niet gekozen.
- **Grootte en transparantie** = urgency: de bubbel groeit en wordt van transparant donkerder naarmate de deadline nadert.
- **Verticale positie** = importance: hoe belangrijker, hoe hoger.
- Een project met deeltaken is een bubbel met kleinere bubbels erin.
- Het dashboard toont standaard de huidige week, met de items op hun exacte due date. Erboven zweven bubbels voor items buiten de week of zonder deadline.
- Change view: volgende week, week, maand of jaar. Filter: alle deadlines, persoonlijk of werk.

## Beslissingen over de functionaliteit

### Taken en projecten

- Je voegt een taak of een project toe. Dat is een keuze los van de datumkeuze.
- Datum: do on (op die dag) of do by (uiterlijk die dag). Een taak kan dus ook een do by hebben.
- Een project heeft deeltaken met eigen due dates. Elke deeltaak is klikbaar en opent een eigen overzicht.
- Eerste versie: project met deeltaken. Deelprojecten in deelprojecten komen later.
- Start-by bij een project: vrij te kiezen, anders standaard vandaag. De echte datum wordt bewaard, niet het woord vandaag. De periode tot de deadline dient als aftelklok.

### Prioriteit

- Elk item heeft een deadline óf een plek op de Eisenhower-matrix. Dit wordt in de frontend én de backend gecontroleerd.
- Importance kiest de gebruiker altijd, op een schaal van 0 tot 100.
- Met deadline wordt urgency berekend uit het aantal dagen tot de deadline. De gebruiker sleept dan alleen omhoog of omlaag.
- Zonder deadline plaatst de gebruiker het item vrij op de matrix (urgency en importance). De stip begint ongeplaatst.
- Bestaande actieve items staan als lichtere cirkels op de matrix, om makkelijk te vergelijken.
- Het Eisenhower-label volgt uit urgency en importance en wordt niet apart bewaard. Een label wijzigen betekent het item verplaatsen.
- De bubbelgrootte gebruikt één schaal voor beide gevallen.

### Voortgang en status

- Voortgang komt uit de deeltaken. Zonder deeltaken schuift de gebruiker de marker zelf.
- Time left wordt getoond als cirkeldiagram.
- Status: ahead of schedule, on schedule, slightly behind, very behind en overdue. Overdue wint zodra de deadline voorbij is en het item niet is afgerond.
- De status vergelijkt voortgang met het verstreken deel van de periode. Urgency gebruikt de absolute dagen tot de deadline. Dat zijn twee verschillende metingen.
- Eenvoudige taken tonen geen voortgang of time left.

### Afronden en archief

- Mark as done zet de status op afgerond, met een afgerond-op-datum. Het item verdwijnt van het dashboard.
- Er is een archiefweergave (filter op status afgerond). Er wordt niets verwijderd.
- Bij afronden vraagt de app of de bijlagen bewaard moeten blijven. Zonder antwoord worden ze bewaard.
- Bijlagen kunnen op elk moment worden toegevoegd of verwijderd.

### Categorieën

- Vaste lijst: werk en persoonlijk, gekozen via een dropdown.

### Zoeken, filteren en pagineren (op de server)

- Zoeken op naam. Filteren op Eisenhower-label, categorie en status.
- Filteren op dag, week, maand of jaar via een datumbereik (van en tot).
- Pagineren met een vaste sortering, bijvoorbeeld op due date.
- Een lijstweergave (bijvoorbeeld archief of alle deadlines) is nodig voor de paginering.

### Bijlagen, plannen en betaling

- Bijlagen staan in Blob Storage. De gegevens erover (gebruiker, bestandsnaam, grootte) staan in SQL.
- Elke gebruiker heeft een totale opslaglimiet: x voor standaard, y voor premium.
- Bij een upload vergelijkt de bijlagen-service de gebruikte ruimte met de limiet van de plan-service. Is de opslag vol, dan toont de frontend een popup (maak ruimte of upgrade naar premium).
- Premium werkt via een gesimuleerde betaalflow met webhook (Mollie of Stripe, de keuze volgt in de lessen).
- Bewaarde betaalgegevens: welke gebruiker, gekozen plan, bedrag, of de betaling gelukt is, betalings-id en tijdstip. De webhook-berichten staan in Cosmos.
- Maand- of jaarabonnement: nog niet beslist.

### Feestdagen en vrije dagen

- Feestdagen komen van een externe API via de backend en staan op een kalenderweergave.
- Gebruikers kunnen persoonlijke vakantiedagen invoeren, met een aanduiding of er die dag gewerkt kan worden.
- Dit komt in een aparte kleine kalender-service. Die is gepland, de definitieve keuze om hem te bouwen volgt.
- Feestdagen worden bewaard in plaats van telkens opgehaald. Valt de externe dienst uit, dan toont de kalender de deadlines zonder feestdagen.

### Admin

- Overzicht van gebruikers met hun plan, en enkele statistieken uit bestaande gegevens (aantal gebruikers, standaard versus premium, aantal aangemaakte items).
- Export alleen voor de admin. Gewone gebruikers krijgen geen export.
- Formaat: CSV als voorstel, nog te bevestigen.
- Activiteit volgen blijft beperkt tot tellingen uit eigen gegevens.

## Architectuur

| Onderdeel | Taak | Opslag |
|---|---|---|
| Identity Server | Login, tokens en rollen user en admin (OAuth en OIDC) | Azure SQL |
| Hoofd-API | Items, deeltaken, filters, afronden | Azure SQL |
| Bijlagen-service | Bestanden, gebruikte ruimte, verwijderen | Blob Storage en SQL |
| Plan-service | Plannen, limieten, betaal-webhook | SQL (plan) en Cosmos (betaalgeschiedenis) |
| Kalender-service (gepland) | Feestdagen via externe API en persoonlijke vrije dagen | SQL |
| React-frontend | Dashboard, formulieren, overzichten | Geen |

Opbouw per service volgens het cursusvoorbeeld: API, domein, opslag, database.

Verbindingen tussen de onderdelen:

- Frontend naar Identity Server (login) en naar de hoofd-API.
- Hoofd-API naar bijlagen-service naar plan-service (opslaglimiet).
- Afronden: de hoofd-API zet de status. De bijlagen-service verwijdert bijlagen op verzoek.
- Betaalprovider naar plan-service via een webhook. De plan-service werkt het plan bij in SQL en bewaart het bericht in Cosmos.
- Kalender-service naar de externe feestdagen-API.

## Hoe we aan de eisen voldoen

### Cloudontwikkeling

| Eis | Invulling |
|---|---|
| Eigen webservices die communiceren | Hoofd-API, bijlagen-service, plan-service, kalender-service (gepland) |
| Externe webservice via de backend | Feestdagen-API via de kalender-service |
| Azure SQL | Gebruikers, items, plannen, vrije dagen, gegevens over bijlagen |
| Cosmos DB | Betaalgeschiedenis (webhook-berichten) |
| Blob Storage | Bijlagen |
| Identity Server (OAuth en OIDC) | Login, rollen user en admin |
| OpenAPI, Scalar en Postman | Documentatie en tests van de API's |
| Beveiligde API met eenvoudige UI | React-frontend met login |

### Web 3

| Eis | Invulling |
|---|---|
| Entiteiten met een 1-N-relatie | Gebruiker naar items, project naar deeltaken |
| Volledige CRUD | Items aanmaken, tonen, wijzigen en verwijderen |
| Minstens twee rollen | User en admin |
| Zoeken, filteren en pagineren op de server | Naam, categorie, periode en status, met een lijst met pagina's |
| Betaalflow met webhook | Premium via gesimuleerde betaling |
| Niet-triviale React context | Ingelogde gebruiker, rol en plan |
| Export | Admin: gebruikers met hun plan, als CSV |
| Toestanden en techniek | Loading, error, leeg en succes (bijvoorbeeld opslag vol). TypeScript zonder any, CORS en CI/CD. |

## Schermen

### Dashboard

- Weekoverzicht met items op hun due date, zwevende bubbels erboven.
- Filter (alles, persoonlijk, werk), change view (week, maand, jaar) en een knop om een nieuw item toe te voegen.
- Klik op een bubbel opent het overzicht van dat item.

### Nieuw item

- Naam, categorie (dropdown), do on of do by, start-by, deeltaken, beschrijving en bijlagen.
- Een matrix met importance en urgency (0 tot 100). Met deadline sleep je alleen omhoog en omlaag, zonder deadline vrij.

### Overzicht van een item

- Naam, categorie, deadline en start, beschrijving, deeltaken met eigen due date (klikbaar).
- Progress bar, time left, statuslabel en bijlagen (toevoegen en verwijderen).
- Knoppen voor edit en mark as done.

## Nog open

- Grenzen tussen slightly behind en very behind.
- Hoe de dagen tot de deadline worden omgezet naar de schaal van 0 tot 100.
- Of de kalender-service effectief wordt gebouwd.
- Welke feestdagen-API en welke betaalprovider.
- Of de betaalprovider meetelt als externe webservice (vraag aan de docent).
- Maand- of jaarabonnement.
- Wie de gegevens voor de admin-export samenbrengt: gebruikers staan in de Identity Server, plannen in de plan-service.
- Wie in de frontend deadlines en vrije dagen samenbrengt, omdat ze uit twee services komen.
- CSV bevestigen als exportformaat.
- Libraries voor de bubbels en de kalender (controleer wat de cursus aanbeveelt).

## Planning

- Projectvoorstel via het formulier: ingediend, één keer voor beide vakken.
- Presentatie van het voorstel: 5 tot 10 minuten met slides, Gent op 27/10 om 10u00. Het is dezelfde presentatie voor Cloud en Web 3.
- Eindverdediging: ongeveer 40 tot 45 minuten voor beide vakken samen, waarschijnlijk in de examenperiode van januari. De datum volgt.

## WBN: toekomstige uitbreidingen

Uit de eerste lijst zijn de aftelklok, het wijzigen van het label en het plaatsen op de matrix intussen onderdeel van het ontwerp.

1. Herhalende taken met onbepaalde interval (ramen wassen om de 6 maanden, boiler jaarlijks, maandelijks rapport) met een done-slider die meteen weer aftelt naar de volgende instantie. Dit is de grootste uitbreiding, omdat een herhalingsregel moet worden bewaard.
2. Recurring reminders, en reminders via een externe API. E-mail is het eenvoudigst. WhatsApp is zwaar. Er is een geplande taak nodig die de deadlines controleert.
3. Snooze voor niet-dringende taken.
4. Werkbare dagen tot een deadline, zonder weekends, feestdagen en vrije dagen waarop niet gewerkt kan worden.
5. Extra categorieën zoals school en andere, en zelf categorieën toevoegen. Categorie wordt dan een eigen entiteit.
6. Deelprojecten in deelprojecten, over meerdere niveaus.
7. Bubbel die ploft bij afronden via rechtsklik. Op touch kan dat via lang indrukken of een knop, met ongedaan maken.
8. Zwevende en animerende bubbels die in real time groeien.
9. Voltooide deeltaken als opgevulde bubbels, zodat de voortgang meteen zichtbaar is.
10. Maand- en jaarabonnement met een geldigheidsdatum.
11. Laatste login en meer statistieken voor de admin.
