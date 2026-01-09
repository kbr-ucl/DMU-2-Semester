# Lægehuset DK

## **Virksomhedsbeskrivelse: Lægehuset DK**

Lægehuset DK er en mindre lægeklinik, der tilbyder almindelige sundhedsydelser til borgere i lokalområdet. Klinikken består af tre praktiserende læger og en sekretær. Deres primære opgaver er at håndtere patientkonsultationer, herunder indgår f.eks.: Vaccinationer, receptfornyelser og generel rådgivning om sundhed.

Klinikken oplever stigende behov for digitalisering, da mange af deres nuværende arbejdsgange foregår manuelt. Tidsbestilling foregår ofte telefonisk, og personalet bruger meget tid op denne opgave.

For at forbedre effektiviteten ønsker Lægehuset DK at få udviklet et simpelt IT‑system, der kan hjælpe med at:

- Registrere patienter og deres stamdata
- Administrere Patientkonsultationer
  - Oprette konsultationsaftaler.
  - Ændre konsultationsaftaler.
  - Slette konsultationsaftaler.
- Give personalet et samlet overblik over dagens konsultationer

Systemet skal være let at bruge, da personalet ikke har teknisk baggrund, og det skal kunne udvides senere, hvis klinikken får behov for flere funktioner.



## Arbejdsprocesser

### Beskrivelse af en konsultationsaftale (med indarbejdet tidsbuffer)

En konsultationsaftale i Lægehuset DK indeholder alle de oplysninger, der er nødvendige for at planlægge et patientbesøg. Aftalen er altid knyttet til én patient og én læge, og den angiver desuden konsultationstypen, f.eks. almindelig konsultation, vaccination, receptfornyelse eller rådgivning. Konsultationstypen bestemmer den forventede varighed, da forskellige ydelser kræver forskellig tid.

Ud over selve konsultationens varighed skal der altid afsættes en fast tidsbuffer på 5 minutter efter hver konsultation. Denne buffer bruges til journalskrivning, oprydning og forberedelse til næste patient. Systemet skal derfor automatisk sikre, at der indlægges et 5-minutters hul mellem alle konsultationer, så kalenderen ikke kan overbookes, og personalet får den nødvendige tid til administrative opgaver.

#### Standardvarigheder for konsultationstyper (inkl. buffer)

| **Konsultationstype**   | **Varighed** | **Fast buffer** | **Samlet tidsblok** |
| ----------------------- | ------------ | --------------- | ------------------- |
| Almindelig konsultation | 20 min       | 5 min           | 25 min              |
| Vaccination             | 10 min       | 5 min           | 15 min              |
| Receptfornyelse         | 10 min       | 5 min           | 15 min              |
| Rådgivning              | 15 min       | 5 min           | 20 min              |



Bemærk at konsultationstypen IKKE skal indeholde buffertiden - den er en systemindstilling.



### Arbejdsproces for konsultationsaftale – Lægehuset DK

**1. Sekretæren opretter aftalen**

- Registrerer patientens stamdata (hvis ikke allerede oprettet).
- Vælger konsultationstype (almindelig konsultation, vaccination, receptfornyelse, rådgivning).
- Hvis patienten ønsker en specifik læge vælges denne.
- Systemet beregner tidsblok (inkl. 5 min buffer) og viser de næste 10 ledige tider (er der ikke valgt en specifik læge er det blandt alle læge).
- Tidsblok vælges og konsultationsaftalen oprettet.

**2. Patienten ankommer**

- Sekretæren markerer patienten som *ankommet* i systemet.
- Status ændres, så lægen kan se, at patienten er klar.

**3. Lægen udfører konsultationen**

- Lægen åbner patientens journal og gennemfører den aftalte konsultation.
- Efter konsultationen bruger lægen bufferen til journalskrivning og forberedelse til næste patient.

**4. Afslutning**

- Systemet registrerer konsultationen som afsluttet.
- Eventuelle opfølgende aftaler oprettes som nye konsultationsaftaler.

