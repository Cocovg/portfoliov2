<div id="top"></div>

# Project X - AR Bloementuin

<div style="display: flex; justify-content: center; gap: 16px; flex-wrap: wrap; margin: 20px 0;">
    <a href="/LO1#project-x" style="display: inline-block; background-color:rgb(219, 52, 52); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO1</a>
    <a href="/LO2#project-x" style="display: inline-block; background-color:rgb(181, 136, 38); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO2</a>
    <a href="/LO3#project-x" style="display: inline-block; background-color:rgb(43, 159, 76); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO3</a>
    <a href="/LO4#project-x" style="display: inline-block; background-color:rgb(33, 75, 225); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO4</a>
    <a href="/LO5#project-x" style="display: inline-block; background-color:rgb(219, 52, 194); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO5</a>
</div>

<div style="display: flex; justify-content: center; gap: 16px; flex-wrap: wrap; margin: 20px 0;">
  <a href="https://github.com/Cocovg/ar-bloementuin" target="_blank" style="display: inline-block; background-color: rgb(88, 52, 143); color: white; padding: 8px 12px; text-decoration: none; border-radius: 4px;">GitHub Repository</a>
</div>

## Projectplan

### Subject
**Titel:** AR Bloementuin

**Korte omschrijving:**
In dit project ontwikkel ik een mobiele Augmented Reality-app waarin gebruikers een virtuele bloementuin maken in hun echte omgeving. De ervaring is aangepast aan de tijd van de dag. De app wordt gebouwd in Unity met AR Foundation en bevat eigen gemaakte 3D-modellen ontworpen in Blender.

**Coach:** Guido

### Content
**Technologieën en tools:**
- Unity 
- AR Foundation (voor ARKit en ARCore)
- Blender (voor 3D-modellen van bloemen, vlinders)
- C# (voor interactie en gedrag)
- Android Studio & Cursor (voor build/testing)

**Belangrijkste functies:**
- Plaatsen van bloemen
- Insecten fladderen door de scène met eenvoudige AI-gedragspatronen
- Performance geoptimaliseerd voor mobiele apparaten

**Doelgroep:**
- Algemeen publiek, met nadruk op gebruikers die creatief zijn en geïnteresseerd zijn in technologie en/of natuur.

### Research

#### Tools
Unity met AR Foundation is een combinatie die veel wordt gebruikt voor AR apps. Het fijne hieraan is, is dat er veel documentatie over is en ikzelf al een klein beetje bekend ben met Unity. 

Unity is ook toegankelijk door de 3D modellen die ik uiteindelijk wil gebruiken. Andere AR software zoals WebAR zijn erg beperkt, WebAR geeft de mogelijkheid om een model te importeren en te bekijken, maar de animaties en functies zijn hevig beperkt en is dus niet geschikt voor mijn project. Deze beperkingen zie ik ook terug in andere software zoals Adobe Aero of Spark AR. De vrijheid die ik met Unity heb, heeft dan ook duidelijk mijn voorkeur.

Voor de codeertaal zal C# worden gebruikt, dit is omdat Unity hier voornamelijk mee werkt en de functies van assets goed ondersteunt. Voor codeertalen zoals Flutter, die vaak bij het bouwen van apps wordt gebruikt, is het lastig en bijna onmogelijk om de 3D functies te gebruiken.

Blender wordt gebruikt voor de 3D models te maken. Deze is gratis, toegankelijk en hier heb ik al wat ervaring mee. De keuze voor deze software was makkelijk gemaakt.

Voor het bouwen en testen van de applicatie gebruik ik Android Studio en als ondersteuning voor AI, Cursor. Android Studio geeft mij een digitale telefoon waar ik de app kan testen voor de UI, al zou ik natuurlijk mijn eigen telefoon moeten gaan gebruiken voor de AR functies.

#### Onderzoeksvragen
1. **Welke uitdagingen ervaart een beginner bij het ontwikkelen van een AR-app met Unity, en hoe kunnen deze worden overwonnen?**
   
   Ik onderzoek mijn eigen leerproces met betrekking tot Unity, C#, AR Foundation en mobiele deployment. Door te reflecteren op technische obstakels en mijn leermethode (zoals AI-hulp, tutorials en documentatie) breng ik mijn ontwikkeling in kaart. – Deliverable: Reflectiedocument – "mijn reis"

2. **Hoe beïnvloedt gebruikersinteractie de beleving van een digitale, natuurlijke AR-omgeving?**
   
   Door feedback te verzamelen van testgebruikers, analyseer ik hoe mensen reageren op interactieve elementen zoals het laten verschijnen van bloemen of het volgen van vlinders. Dit geeft inzicht in wat werkt en hoe de ervaring wordt beleefd. – Reeks usertests

### Leerdoelen
- Ik maak een design voor een interactieve app (LO1)
- Ik leer hoe je met Unity en AR Foundation een mobiele AR-applicatie ontwikkelt (LO3)
- Ik begrijp hoe touch-interactie werkt in een AR-omgeving (LO2)
- Ik oefen met het animeren en plaatsen van 3D-modellen in AR (LO2)
- Ik leer reflecteren op mijn leerproces en technische keuzes (LO5)
- Ik leer hoe gebruikers reageren op een natuurlijke, visuele AR-ervaring (LO4)

### Planning (4 weken)

<table style="width: 100%; border-collapse: collapse; margin: 20px 0; font-family: Arial, sans-serif;">
  <thead>
    <tr>
      <th style="padding: 15px; text-align: left; border-bottom: 2px solid #dee2e6; font-weight: bold;">Week</th>
      <th style="padding: 15px; text-align: left; border-bottom: 2px solid #dee2e6; font-weight: bold;">Activiteiten</th>
      <th style="padding: 15px; text-align: left; border-bottom: 2px solid #dee2e6; font-weight: bold;">Deliverables / Doelen</th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom: 1px solid #dee2e6;">
      <td style="padding: 15px; vertical-align: top; font-weight: 500;">Week 1<br>Opstart & Onderzoek</td>
      <td style="padding: 15px; vertical-align: top;">
        - Projectopzet & definitieve scope bepalen<br>
        - Installeren Unity + AR Foundation setup<br>
        - Eenvoudige AR-demo werkend krijgen<br>
        - Eerste Blender-modellen maken
      </td>
      <td style="padding: 15px; vertical-align: top;">
        - Werkende AR-basis<br>
        - Eerste 3D-modellen gemaakt<br>
        - Documentatie projectdoel
      </td>
    </tr>
    <tr style="border-bottom: 1px solid #dee2e6;">
      <td style="padding: 15px; vertical-align: top; font-weight: 500;">Week 2<br>Modelling & Interactie</td>
      <td style="padding: 15px; vertical-align: top;">
        - Animaties maken (bloei, vleugels)<br>
        - In Unity: interactie via tap<br>
        - Vlinderbeweging + AI-patronen
      </td>
      <td style="padding: 15px; vertical-align: top;">
        - Bloemen kunnen worden geplant via touch<br>
        - Vlinders bewegen rond<br>
        - Visueel eerste indruk bloementuin
      </td>
    </tr>
    <tr style="border-bottom: 1px solid #dee2e6;">
      <td style="padding: 15px; vertical-align: top; font-weight: 500;">Week 3<br>Sfeer & Functionaliteit</td>
      <td style="padding: 15px; vertical-align: top;">
        - UX verbeteren (UI-knopjes, hints)
      </td>
      <td style="padding: 15px; vertical-align: top;">
        - Rustgevende omgeving compleet
      </td>
    </tr>
    <tr>
      <td style="padding: 15px; vertical-align: top; font-weight: 500;">Week 4<br>Testen & Oplevering</td>
      <td style="padding: 15px; vertical-align: top;">
        - Testen op meerdere devices<br>
        - Bugfixes & performance optimalisatie<br>
        - Documentatie schrijven<br>
        - Video/demo opnemen
      </td>
      <td style="padding: 15px; vertical-align: top;">
        - Werkende app-demo<br>
        - Eindrapport + projectpresentatie<br>
        - Reflectie op leerdoelen
      </td>
    </tr>
  </tbody>
</table>

<h2 id="concept">Concept</h2>

![concept](/img/proX/con-1.png)

Om te beginnen wilde ik een perspectief duidelijk uittekenen. Hier loopt een persoon met een telefoon door een stad en kan deze vullen met een aantal bloemen wanneer de persoon op het scherm tikt. Daarnaast leek het me leuk om op tijd gebaseerde insecten te hebben en eventueel de planten ook te laten bloeien wanneer deze worden geplaatst.

<h2 id="ui">UI</h2>

![concept](/img/proX/con-2.png)

Voor de UI had ik verschillende conceptuele versies gemaakt. Hier was de uitdaging voornamelijk om het scherm heel leeg te houden maar nog steeds een duidelijke locatie voor instellingen te geven. Zo is het belangrijk dat het scherm altijd de ruimte heeft om de omgeving goed te kunnen bekijken en bloemen gemakkelijk te kunnen plaatsen waar die dat wilt.

![concept](/img/proX/con-1.png)
 
De conceptuele uitwerking was ook meer gericht naar 2 kleine cirkelvormen linksonder in de hoek. Dit oogt minimaal maar nog steeds duidelijk doordat deze aan de linkerkant zit. Wanneer op deze knoppen wordt gedrukt, opent dit een groter menu waar een gebruiker overzichtelijk de instellingen kan bekijken.

<h2 id="logo">Logo</h2>


![concept](/img/proX/logo-1.png)

Voor de logo's was ik ook op zoek naar een benaming die bij de app zou passen. In het gehele project heb ik gerefereerd naar de naam GARDEN. Deze naam trok mij aan omdat je doormiddel van de app, elke plek jouw tuin kan maken.

<h2 id="3d-model">3D model</h2>

Om de tuin te vullen had ik in ieder geval een enkele bloem nodig. Deze bloem heb ik gemaakt in Blender met behulp van verschillende tutorials:

- [How to Circular Array in Blender | Radial Array Tutorial - TLD Studios](https://www.youtube.com/watch?v=CxvZv2yg-Ls)
- [Create a Flower in 1 Minute in Blender - Beginner Tutorial - 3D Artifex Hub](https://youtube.com/shorts/jGKj7P1CIOk?si=AvB9TPfCEtC4IMKN)

Hier heb ik geleerd de bloemenblaadjes te vormen in een lichte boging, met daarbij een automatische herhaling van de vorm in een circulaire vorm. Zo kon ik de data van mijn blaadjes behouden, daarnaast door de herhaling was ik snel klaar en hoefde ik niet lang en ongemakkelijk de blaadjes heel precies te plaatsen.

![concept](/img/proX/3d-1.png)

<h2 id="voorbereiding">Voorbereiding</h2>

<h3 id="opzet">Opzet</h3>

- [Get Started with AR Foundation in Unity - Beginner Friendly Tutorial 2023 - samyam](https://www.youtube.com/watch?v=FWyTf3USDCQ)

Voordat ik kon beginnen met het ontwikkelen, moest ik mijn Unity omgeving aanpassen aan een AR app. Zo heeft Unity een template beschikbaar die al een aantal onderdelen klaarzet en download.
Zo was het belangrijk sommige extensions actief te hebben, zoals GoogleAR en AR Foundation. Deze ondersteunen de template en zijn nodig om een Android AR app te kunnen bouwen.

![concept](/img/proX/omg-1.png)

<h3 id="tegenslagen">Tegenslagen</h3>

Mijn grootste en meest problematische tegenslag kwam al vrij snel. Voor mijn laptop was het niet mogelijk om Android Studio te gebruiken. Deze werd niet ondersteund en was onmogelijk te gebruiken. Android Studio geeft de mogelijkheid om via je systeem een "neppe" telefoon te gebruiken zodat je direct en gemakkelijk je app functies kunt uittesten. Daarbij zijn problemen als errors en waarschuwingen snel zichtbaar en direct op te lossen.

Als vervanging heb ik continu de app "gebuild", waarbij ik langere periodes moest wachten om het resultaat te zien van wat ik had gemaakt. Dit was in de eerdere stappen nog best "oké" te doen. Maar nadat er meer fouten in mijn mobiele console kwamen, kon ik steeds moeilijker problemen oplossen. Zo was de tekst extreem klein en moest ik problemen met een beetje een zeggende tekst handmatig overtypen en proberen uit te vogelen.

Wat mij uiteindelijk ook tegenzat was versiebeheer. Unity was toch even anders werken en gaat maar een klein aantal stappen terug wanneer je handeling probeert ongedaan te krijgen. Dus ik verloor vaak werkende builds doordat ik niet genoeg ongedaan kreeg. Daarnaast snapte ik niet helemaal hoe ik op GitHub het versiebeheer daar kon regelen. Iets wat ik zeker beter even had kunnen opzoeken.


![concept](/img/proX/pro-1.png)

<h2 id="prototype-1">Prototype 1</h2>

Voor mijn eerste prototype begon ik eerst met het importeren en plaatsen van de bloemen. Zo had ik mijn omgeving opgezet en ben ik doormiddel van onderstaande video's door de eerste stappen gekomen:

- [Tap to Place Objects in Unity's AR Foundation - Tutorial 2023 - samyam](https://www.youtube.com/watch?v=lYDfV-GaKQA&t=1s)
- [Unity 6 Tutorial: Building Your Augmented Reality App to Device - Cam Ayres](https://www.youtube.com/watch?v=0uErb9NSLUs)


Met deze videos kreeg ik de basiskennis over de GameObjects in de zijbalk en kon ik simpel een bloem in de scene plaatsen, die vervolgens te zien was in AR.

Hierna ben ik bezig geweest met het zelf plaatsen van de bloem. Waar ik met behulp van AI en bronnen een stuk vooruit was gekomen. Zo kon ik steeds een enkele bloem plaatsen maar werd de vorige verwijderd. Doormiddel van mijn code aan te passen in het bestand waar ik aangeef bloemen te willen plaatsen, heb ik de restrictie weggehaald van een enkele bloem. Daarna heb ik aangegeven meerdere bloemen te willen plaatsen in een ruimte en dit loste het probleem snel op.

![concept](/img/proX/pro-6.png)
![concept](/img/proX/pro-2.png)

Wat mij snel opviel is de grootte van de bloemen, deze waren enorm. Hier moest ik terug Blender in om het model te verkleinen.

Naast dat ik de bloemen had aangepast in grootte, heb ik ook de kleuren aangepast zodat de bloemen op madeliefjes lijken. In deze versie viel mij alleen op dat de schaduwen wel heel dramatisch geplaatst werden. Zo probeerde ik ook een versie uit waar ik de lichtbron uit de scene haalde, helaas werden de bloemen hier heel plat en saai.

![concept](/img/proX/pro-3.png)

Uiteindelijk heb ik dit prototype vrij stuk gemaakt toen ik een startmenu wilde maken die gebruikers de app lieten starten of beëindigen. De "planes" van de app waren niet meer te zien en de bloemen konden niet meer geplaatst worden.

<h2 id="prototype-2">Prototype 2</h2>

In mijn 2e prototype probeerde ik de app te bouwen zodat ik begon met het menuscherm en daarna pas de AR omgeving, mocht dit het probleem van het vorige prototype verhelpen. Snel kwam ik er wel achter dat ik precies dezelfde problemen tegenkwam, het menuscherm werkte vlekkeloos maar ik kon vervolgens niet mijn functies uitvoeren in de AR omgeving. 

Het probleem begon zo complex te worden dat ik er met AI ook niet meer uitkwam. Door beide projecten erbij te pakken vond ik wel echt dat het probleem voornamelijk bij de handelingen in het menuscherm lag, maar ben nooit erachter gekomen wat precies fout ging. Zo lag AI mij alles uit en ging door alle instellingen van de GameObject, vond ik helaas niks dat afweek.

Na wat camera instellingen te wijzigen, kreeg ik wel mijn planes terug en kon ik een enkel object plaatsen. Maar waren de planes niet voldoende gekoppeld aan mijn omgeving en bleven deze vaak hangen aan een vaste locatie op mijn scherm.


![concept](/img/proX/pro-4.png)
![concept](/img/proX/pro-5.png)

<h2 id="reflectie">Reflectie</h2>

Dit project was voor mij een waardevolle leercurve in AR-ontwikkeling. Ondanks dat het eindresultaat niet volledig aan mijn verwachtingen voldeed, heb ik veel geleerd over Unity, 3D-modellering in Blender en de uitdagingen van AR-ontwikkeling.

De grootste les was het belang van een goede ontwikkelomgeving en debug-tools. Door de problemen met Android Studio werd het testen en debuggen onnodig gecompliceerd, wat veel tijd kostte. In toekomstige projecten zou ik eerst zorgen voor een stabiele en goed werkende ontwikkelomgeving.

Verder heb ik geleerd dat versiebeheer in Unity anders werkt dan ik gewend was, en dat dit cruciaal is bij het ontwikkelen van complexe applicaties. Het regelmatig maken van backups en het beter begrijpen van GitHub voor Unity-projecten zijn vaardigheden die ik in de toekomst wil verbeteren.

Qua ontwerp ben ik tevreden met de UI-concepten en het 3D-model van de bloem. Deze elementen sluiten goed aan bij mijn visie voor de app en tonen aan dat ik de ontwerpprincipes begrijp die nodig zijn voor een intuïtieve AR-ervaring.


