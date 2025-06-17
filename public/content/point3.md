<div id="top"></div>

# Cardan Development 

<div style="display: flex; justify-content: center; gap: 16px; flex-wrap: wrap; margin: 20px 0;">
    <a href="/LO1#development-cardan" style="display: inline-block; background-color:rgb(219, 52, 52); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO1</a>
    <a href="/LO3#development-cardan" style="display: inline-block; background-color:rgb(43, 159, 76); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO3</a>
    <a href="/LO4#development-cardan" style="display: inline-block; background-color:rgb(33, 75, 225); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO4</a>
    <a href="/LO5#development-cardan" style="display: inline-block; background-color:rgb(219, 52, 194); color: white; padding: 3px 8px; border-radius: 4px; font-weight: bold; margin-right: 5px; text-decoration: none;">LO5</a>
</div>

<div style="display: flex; justify-content: center; gap: 16px; flex-wrap: wrap; margin: 20px 0;">
  <a href="https://www.figma.com/design/KIVWaDCLVYXIBWR4YJ13M1/Project-Development?node-id=2-2&t=OAVSoPqAwt6y49og-1" target="_blank" style="display: inline-block; background-color: #4a54f1; color: white; padding: 8px 12px; text-decoration: none; border-radius: 4px;">Figma omgeving</a>
  <a href="/files/development procesverslag.pdf" target="_blank" style="display: inline-block; background-color:rgb(60, 154, 186); color: white; padding: 8px 12px; text-decoration: none; border-radius: 4px;">Procesverslag PDF</a>
   <a href="https://github.com/Cocovg/Cardan" target="_blank" style="display: inline-block; background-color:rgb(88, 52, 143); color: white; padding: 8px 12px; text-decoration: none; border-radius: 4px;">GitHub project</a>
   <a href="https://cardan.cocovanglabbeek.nl/" target="_blank" style="display: inline-block; background-color:rgb(66, 143, 50); color: white; padding: 8px 12px; text-decoration: none; border-radius: 4px;">Live website</a>
</div>

## Project beschrijving
In dit project houden we ons bezig met het ontwikkelen van de functie die we in het vorige project hebben ontwikkeld als design. De functie wordt hier werkend en verwerkt de feedback die is gekregen tijdens de oplevering aan de klant. 
In dit project werkte ik samen met Laurens Kosters en hebben we ons gericht op een functie die vanuit de website een filter kan geven aan een bestaande website of een prefab website die door ons is gemaakt.
 
## Communicatie
Voor communicatie hebben we voornamelijk Discord en GitHub gebruikt. In Discord deelde we bronnen, werkomgevingen en notities. 

 ![Figma overview](/img/development/com-1.png)

 
In onze GitHub omgeving, hadden we een project staan waar we onze iteraties en proces konden bijhouden. Doormiddel van meerdere commits konden we meerdere versies bijhouden van elke functie.

 ![Figma overview](/img/development/com-2.png)
 
Daarnaast hebben we gebruikt gemaakt van het project onderdeel waarbij wij taken konden verdelen en bijhouden. Vanuit hier konden wij branches aanmaken per taak/functie. Dit hielp ons om geordend te blijven en een goed overzicht te behouden over het project.

 ![Figma overview](/img/development/com-3.png)
 
Zoals de foto hieronder, zijn de branches gebruikt om de main branch aan te vullen met de functies tot een geheel product. Deze zijn gelinkt aan de taken die wij in het project hier boven gebruikte en konden direct via git de branches aanmaken en mergen met de main branch

 ![Figma overview](/img/development/com-4.png)
 
<h2 id="design">Design</h2>
In het vorige project was al een groot deel van het design gemaakt. Dus heb ik mij deze fase gericht op met verwerken van feedback in het design van de "Blob". Hier kreeg ik tijdens de eindpresentatie van het UX project complimenten over het idee en de functie van de blob, maar mocht deze nog meer opvallen. Vanuit de usertests kwam dit ook naar voren en kreeg ik meer feedback over de kleur en de locatie. Zo heb ik met meerdere vormen gespeeld en heb ik variaties met kleur uitgeprobeerd. Van Bram kreeg ik tijdens dit project ook de feedback om hem eventueel schuin te plaatsen. Zelf was ik daar niet zo een fan van omdat ik het niet helemaal bij de rest van de pagina en stijl vond passen en daarnaast wilde ik dat de blob niet als een sticker werd gezien. Daarbij heb ik later in het proces een extra functie voor deze knop bedacht, waar het idee van een sticker niet meer bij zou aansluiten. Omdat deze functie meer beweging veroorzaakt.
 ![Figma overview](/img/development/des-1.png)

## Development
<h3 id="voorpagina">Voorpagina</h3>
Voor development ben ik zelf begonnen met het opzet van de beginpagina. Deze is nagemaakt met V0.dev. Hier was het mogelijk een screenshot van een bestaande pagina te laten kopiëren, wat vervolgens mij de code gaf om zelf nog aan te passen. Zo heb ik zelf nog de kleuren aangepast naar de officiële Cardan kleuren, de foto aangepast en font en lettergrootte aangepast. De pagina is ook niet volledige 1 op 1, dit was niet volledig nodig voor de ontwikkeling van de website. Wel kan het de klant een goed en realistisch beeld geven over hoe de functie eruitziet op hun pagina. Hieronder is het beeld van de AI en mijn uiteindelijke versie te zien.

![Figma overview](/img/development/dev-1.png)
![Figma overview](/img/development/dev-2.png)

<h3 id="call-to-action-knop">Call-to-action knop</h3>

Vervolgend ben ik gaan werken aan de primaire knop en de functie. In het project noemde wij deze de "blob". Deze is qua design veranderd tijdens deze fase en heeft ook een extra functie gekregen. Daarnaast heb ik van de blob ook een component gemaakt zodat deze gemakkelijk kan worden aangepast zonder te hoeven zoeken naar de bijbehorende code. Daarbij ook is de component gemakkelijk te gebruiken in andere projecten door de component daarin te zetten.

![Figma overview](/img/development/dev-3.png)
![Figma overview](/img/development/dev-4.png)
![Figma overview](/img/development/dev-5.png)
     

Zo is de blob statisch en scrolt mee met de pagina. Als nieuwe functie beweegt de blob ook wanneer je misklikt op de knop. Wat een gevoel moet geven van irritatie. Iets wat een persoon met een beperking vaak ervaart. 

![Figma overview](/img/development/dev-6.png)
![Figma overview](/img/development/dev-7.png)
    
### Responsive

Ook heb ik gekeken naar de responsiveness van de knop. Deze moest natuurlijk een stuk kleiner en nogsteeds de functie kunnen behouden. Waar ik als eerste tegen aanliep is de informatie in de blob, dit was niet goed te lezen en daarbij veel te veel voor in een beperkte plek. Zo heb ik alleen de titel behouden en de knop, wat mij al meer opties gaf. Ook heb ik een sluitknop toegevoegd zodat de website nogsteeds prettig kan worden bekeken zonder dat de blob veel informatie zou bedekken.

![Figma overview](/img/development/dev-8.png)

<h3 id="prefab-website">Prefab website</h3>
 
Ten slot ben ik bezig geweest met het maken van de prefab website, de bloemenwinkel. Vanuit de feedback van Carolina, onze stakeholder, heb ik deze zo gemaakt dat deze bijna helemaal op de winkel van Coppelmans lijkt. Carolina vertelde ons dat het haar leuk leek als de website heel confronterend kon zijn. Daarnaast is het ook realistisch omdat de website daadwerkelijk bestaat en bedrijven dus ook echt de fouten maken die ontoegankelijk worden ervaren. Ook dit initiële design heb ik ook V0 gebruikt. Al heb ik al het beeldmateriaal naar Coppelmans verwijderd en nieuwe toegevoegd. Voor het beeldmateriaal heb ik gelet op kleuren en variatie. Ook heb ik bij de website naar benamingen van producten bekeken om deze zo onhandig mogelijk te maken, door geen kleuren te vernoemen. Dit maakt het vrijwel onmogelijk om we weten wat voor kleur een plant heeft. De website is te bedienen door de producten te bekijken, die je vervolgens op de productenpagina kunt filteren, wat in verschillende modus van de filter niet gebruiksvriendelijk is.    
![Figma overview](/img/development/dev-9.png)
![Figma overview](/img/development/dev-10.png)

 
## Testen {#testen-2}
### Doel van mijn test
Voor deze test wilde ik onderzoeken hoe een gebruiker de kleurenblindheidssimulatie ervaart wanneer hij of zij uiteindelijk bij de producten van een bloemenwinkel uitkomt. Ik was benieuwd hoe intuïtief de navigatie is, of het proces helder verloopt en waar eventuele verwarring ontstaat. Ook wilde ik kijken of de filters goed werken en of er visuele of functionele obstakels zijn tijdens het gebruik van mijn ontwerp.

---

### Situatie voor de tester
"Je bent iemand die geïnteresseerd is in kleurenblindheid en je wilt ervaren hoe een persoon met kleurenblindheid een website bekijkt. Je bent op de website van Cardan en probeert deze simulatie uit. Uiteindelijk wil je bij de producten van een bloemenwinkel terechtkomen."
Ik observeerde hoe de tester de stappen doorliep en stelde na afloop enkele reflectievragen over gebruiksgemak, verwachtingen en eventuele frustraties.

---

### Testverloop
•	De eerste stappen van de website werden vlot doorlopen. Alles werd snel gevonden, zonder momenten van verwarring of stilstand.
•	Bij het openen van de bloemenwinkel begon automatisch een tutorial. De tester probeerde meteen interactie met de tutorial, maar dat werkte nog niet.
•	Hij begreep echter direct dat de tutorial eerst afgemaakt moest worden voordat hij verder kon, wat positief was.
•	Toen hij de kleurenblindheidsfilters begon te testen, was hij positief verrast over het effect en hoe goed het werkte.
•	Ik liet hem ook zijn eigen website invoeren om te testen of de simulatie daar werkte. We probeerden meerdere pagina's, maar op de meeste lukte het niet om de filter correct over de website te plaatsen.

---

### Belangrijke observaties
•	Het was niet mogelijk om meerdere beperkingen tegelijk te activeren, bijvoorbeeld kleurenblindheid én kokervisie. Dat werd als een beperking ervaren.
•	De filterbalk geeft visuele highlights aan wanneer een filter actief is, maar die waren voor hem niet goed zichtbaar. Dit kan aan persoonlijke perceptie of aan het beeldscherm liggen.

---

### Algemene feedback en inzichten
De test gaf mij het inzicht dat de ervaring over het algemeen duidelijk en direct was. De gebruiker begreep intuïtief hoe de tool werkt en vond het proces prettig. Toch zijn er enkele gebruiksvriendelijkheidspunten die ik kan verbeteren.

---

### Verbeterpunten op basis van deze test

<table style="width: 100%; border-collapse: collapse; margin: 20px 0; font-family: Arial, sans-serif;">
  <thead>
    <tr>
      <th style="padding: 15px; text-align: left; border-bottom: 2px solid #dee2e6; font-weight: bold;">Onderdeel</th>
      <th style="padding: 15px; text-align: left; border-bottom: 2px solid #dee2e6; font-weight: bold;">Verbeterpunt</th>
    </tr>
  </thead>
  <tbody>
    <tr style="border-bottom: 1px solid #dee2e6;">
      <td style="padding: 15px; vertical-align: top; font-weight: 500;">Filterfunctionaliteit</td>
      <td style="padding: 15px; vertical-align: top;">Mogelijkheid overwegen om meerdere filters tegelijk te activeren</td>
    </tr>
    <tr style="border-bottom: 1px solid #dee2e6;">
      <td style="padding: 15px; vertical-align: top; font-weight: 500;">Filterbalk</td>
      <td style="padding: 15px; vertical-align: top;">Actieve status visueel duidelijker maken (bijvoorbeeld door sterkere kleuren of animatie)</td>
    </tr>
    <tr style="border-bottom: 1px solid #dee2e6;">
      <td style="padding: 15px; vertical-align: top; font-weight: 500;">Gebruik op externe websites</td>
      <td style="padding: 15px; vertical-align: top;">Niet alle websites ondersteunen het filter – ik overweeg een disclaimer toe te voegen bij deze functionaliteit</td>
    </tr>
    <tr>
      <td style="padding: 15px; vertical-align: top; font-weight: 500;">Tutorial</td>
      <td style="padding: 15px; vertical-align: top;">Interactie pas mogelijk maken ná afronden tutorial – dit was nu verwarrend</td>
    </tr>
  </tbody>
</table>

## Reflectie

Bij dit project heb ik relatief weinig moeite ervaren. Dit kwam deels door de samenwerking met Laurens, maar ook doordat het niveau goed aansloot bij mijn huidige kennis. Dankzij mijn voorkennis kon ik met vertrouwen en gemak aan het project werken. Bovendien waren de scope en het doel van het project al duidelijk, mede door ervaringen uit het vorige project. De ontwikkeling verliep vlot, mede dankzij het gebruik van AI, wat het proces aanzienlijk versneld en vereenvoudigd heeft.

Het werken met GitHub heeft mij geholpen om efficiënter te werken. We hebben een taakverdeling gemaakt die aansloot bij ons niveau en beschikbare tijd, wat het mogelijk maakte om flexibel te blijven tijdens de uitvoering. Door het gebruik van meerdere branches en commits konden we gemakkelijk teruggrijpen op eerdere versies wanneer bepaalde functies niet naar verwachting werkten.

Ook de communicatie binnen het team verliep uitstekend. Laurens en ik konden op een prettige en effectieve manier met elkaar communiceren, wat het eenvoudiger maakte om feedback te geven en te ontvangen. We waren daarnaast in staat om elkaars taken op te vangen wanneer nodig. Met name bij het gebruik van Git heeft Laurens mij goed ondersteund wanneer ik er even niet uitkwam.

Voor toekomstige projecten wil ik mijzelf meer uitdagen op het gebied van softwareontwikkeling. In dit project heb ik, uit praktische overwegingen, veel gebruikgemaakt van AI. Hoewel dit functioneel was, had ik het in sommige gevallen ook zonder deze ondersteuning kunnen oplossen. Daarom wil ik bij een volgend project bewuster kiezen voor eigen ontwikkeling, om zo mijn kennis en vaardigheden verder te verdiepen.