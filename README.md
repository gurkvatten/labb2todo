Appen är ett Kanban-board för sidoprojekt med en unik funktion: den hjälper dig att identifiera projekt du glömt bort. Varje uppgift har en updatedAt-tidsstämpel som uppdateras vid skapande eller när den flyttas mellan kolumner. Appen använder denna information för att beräkna hur många dagar sedan ett projekt senast uppdaterades.

Denna beräkning styr en style binding (:style) på korten, vilket gör att de bleknar ut ju längre sedan de senast uppdaterades. Aktiva projekt förblir tydliga medan försummade projekt gradvis sjunker bakåt visuellt. Samma logik används för att visa ”mest bortglömda projekt” i inställningspanelen (styrd med v-if) och visas även i projektfiltret där antalet dagar sedan aktivitet visas direkt i dropdownen.

Utöver dessa funktioner erbjuder appen:

	•	 Filtrering per projekt (v-model + computed)
	•	 Prioritetsfärg på korten via style binding
	•	 Drag-and-drop mellan kolumner
	•	 Dark/light-läge, sparat med localStorage

Drag-and-drop funktionaliteten använder webbläsarens inbyggda HTML5 Drag & Drop (draggable, @dragstart, @dragover.prevent, @drop) istället för ett bibliotek. Detta val gjordes främst för att undvika en extra dependency och för att få en djupare förståelse för hela flödet, inklusive att spara den dragna uppgiften, hantera drop-zonen och uppdatera state.

En nackdel med att använda HTML5 Drag & Drop är dess begränsade funktionalitet på touch-enheter (mobil/surfplatta), vilket är en nackdel med tanke på appens responsiva design. Om tiden hade tillåtit hade jag bytt till vuedraggable, som erbjuder samma funktionalitet med touch-stöd och förbättrade animationer.
