# Homewizard Plug-in batterij dynamisch (ont)laden 

## Waarom
De [Homewizard Plugin-In thuisbatterij](https://www.homewizard.com/nl/plug-in-battery/) is net gelanceerd in Nederland. Vooralsnog is deze bedoeld om Nul op de Meter (NOM) te draaien. Via P1 uit de slimme meter ziet de [Homewizard P1 Meter](https://www.homewizard.com/nl/p1-meter/)-dongle hoeveel overproductie er is, en draagt de batterij op om dit overschot op te laden. Als er afname is van het stroomnet, zal deze juist ontladen. 
Dit houdt (vooralsnog 9-3-25) geen rekening met dynamische energietarieven.

## Wat:
Het idee is dat als je de P1 informatie die de P1 Meter ontvangt kan controleren, dat je indirect de batterij kan sturen. 
Idealiter kun je dit via Home Assistant aansturen voor automatische optimalisatie. Op deze manier kun je de batterij respectievelijk opladen en ontladen tijdens goedkope en dure dynamische uurtarieven. 
Zolang er nog gesaldeerd kan worden (en je teruglevering lager is dan gebruik), kun je op deze manier de cycles van de batterij financieel het meest voordelig inzetten.

![Smart-stuff P1 meter foward naar Homewizard P1 meter](images/smartstuff-p1+homewizrd-p1.jpeg "P1's")

## Hoe:
Ik gebruik een P1 dongle van Smart Stuff (vorige versie van https://smart-stuff.nl/product/p1-dongle-watermeter/) om mijn slimme meter en watermeter uit te lezen. Deze kan draaien op ESPHome en bevat ook een P1-poort waar de gegevens op worden geforward. 
Hierin zit mijn HW P1 meter. In plaats van letterlijk forwarden van het P1-telegram, herschrijf ik levering en teruglevering voordat deze naar de HW P1 meter gaat. Zie `p1-dongle-pro-h2o-v2-virtual-p1.yaml` voor de broncode. Het is niet de beste C++, maar met Copilot kreeg ik het werkend.

Aangezien ESPHome goed integreert met Home Assistant is het daarna kwestie van aansturen op het wattage (`number.virtual_p1_power`) op een slimme manier te regelen.
Hiervoor heb ik een `input_numer.plug_in_battery_target_power` sensor en een automation gemaakt die de virtuele P1 meter aanstuurt. De smart-stuff dongle zet deze waarden door naar de P1 meter van Homewizard die de batterij weer aanstuurt.
Dit is opgzet is `packages/plug_in_battery.yaml`, welke je in de gelijknamige directory in de Home Assistant `config` directory kunt plaatsen.

## Vervolg
Mijn vervolgstap is om met behulp van [Cheapest Energy Hours](https://github.com/TheFes/cheapest-energy-hours) en een automation de batterij automatisch op de 3,5 goedkoopste en duurste uren te laten laden en ontladen.

## Demo

https://vimeo.com/1064030293/3f3f958084

![Controle vanuit HA](images/charge-kiezen-in-HA.png)

![HW laad cyclus](images/HW-overview.png)

## Opmerkingen

* HomeWizard Energy statistieken slaan nergens meer op.
* Het heeft pas zin om terug te leveren als je meer dan €0.088 per kWh kunt "verdienen". Dit is de Afschrijving per kWh: €1425 (batterij+P1 meter) / 6000 Batterijcycli/ 2.7 kWh capaciteit = €0,088 . Het prijsverschil tussen laden/ontladen zal dus minstens 8,8ct moeten zijn, om te minste de afschrijving van de batterij kiet te spelen.
* C++-code voor bewerken P1-telegrammen kan nog worden verbeterd/versneld.
* Gebruik eventueel [Device Tools](https://community.home-assistant.io/t/device-tools-create-modify-and-merge-devices/691172) om de sensoren aan de P1 meter en Batterij device te koppelen.