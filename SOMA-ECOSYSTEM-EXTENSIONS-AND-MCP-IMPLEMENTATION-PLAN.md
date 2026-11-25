# SOMA – celovit načrt razširitev, Skills, MCP in povezovalnikov

**Raziskava in načrt: 25. november 2026**  
**Ciljni programski repozitorij:** [robertpezdirc8-prog/SOMA](https://github.com/robertpezdirc8-prog/SOMA)  
**Ta dokument je shranjen v:** [robertpezdirc12-design/soma-website](https://github.com/robertpezdirc12-design/soma-website)  
**Povezani produkt:** [Draftwright](https://github.com/robertpezdirc12-design/draftwright-demo)

---

# NAVODILO ZA CODING AGENTA

To je glavno raziskovalno in izvedbeno navodilo za razširitev SOMA v čim bolj zmogljivo, razširljivo AI-CAD okolje. Ne gre za nalogo, da se v projekt kopira čim več GitHub repozitorijev. Cilj je izbrati najboljše preverjene komponente, jih smiselno povezati s SOMA, razširiti CAD-zmožnosti in AI-znanje ter zagotoviti, da so nove funkcije dejansko uporabne in preizkušene.

**Najprej opravi pregled in pripravi načrt. Kode ne spreminjaj, dokler ni končana inventura, ocena združljivosti, varnosti in licenc.** Po pregledu izvajaj spremembe na ločeni veji SOMA. Ne delaj sprememb programske kode v tem spletnem repozitoriju, razen dokumentacije spletnega mesta, ki je izrecno zahtevana. Dejanska aplikacija je v repozitoriju SOMA.

## Kaj želimo doseči

SOMA naj postane razširljiv AI-CAD delovni sistem, ki združuje:

1. **Pravo parametrično CAD-modeliranje** – CadQuery/OpenCascade ostane primarno geometrijsko jedro.
2. **Večjo knjižnico konstrukcijskih elementov** – profili, spoji, pritrdilni elementi, standardni deli, materiali in podsestavi.
3. **Strokovne AI Skills** – pravila in preverjeni postopki za CAD, strojništvo, proizvodnjo, risbe, tolerance, sestave, 3D-tisk, CNC in analize.
4. **MCP in povezovalnike** – varno upravljanje SOMA iz Codex, Claude Code, Gemini CLI, Cursor in drugih združljivih agentov.
5. **Merljivo geometrijsko preverjanje** – model mora biti ne samo lep na sliki, ampak geometrijsko veljaven, dimenzijsko pravilen in preverjen v izvoznih formatih.
6. **Zunanjo interoperabilnost** – po potrebi povezava s FreeCAD, Autodesk Fusion, KiCad, PartCAD in drugimi sistemi brez podvajanja osnovnega CAD-jedra.
7. **Povezavo z Draftwright** – Draftwright razume fotografijo, skico, meritve in posebna pravila izdelka; SOMA iz teh podatkov izdela dejansko parametrično 3D-geometrijo.

---

# 1. Najprej preveri trenutno stanje SOMA

Ne zanašaj se samo na ta dokument ali README. Preglej dejanski trenutni main, vse relevantne mape, zadnje commite, odprte PR-je, teste in lokalne spremembe.

## 1.1 Trenutna osnova, ki jo je treba ohraniti

V pregledani različici SOMA je bila ugotovljena naslednja arhitektura. Vse različice pred izvedbo ponovno preveri:

| Področje | Pregledana osnova |
|---|---|
| Namizna aplikacija | Electron + TypeScript |
| 3D prikaz | three.js |
| CAD jedro | CadQuery + OpenCascade/OCP |
| Python runtime | 3.12.15 |
| CadQuery | 2.8.0 |
| CAD model | Python parametrična koda |
| CLI | soma build, soma export, soma analyze, soma drawing, soma anim in drugi ukazi |
| Datoteka projekta | .soma |
| Parametri | uporabniški panel in param() |
| FEM | CalculiX in obstoječa implementacija FEM |
| Tehnične risbe | PDF/DXF in risalno jedro SOMA |
| Izvozi | STEP, STL, 3MF, AMF, BREP, IGES, OBJ, VRML, GLB, GLTF, DXF, SVG |
| AI agent | SOMA AI ter zunanji CLI agenti |
| Skills | vgrajene Markdown datoteke in upravljana knjižnica lastnih Skills |

Preveri različice in združljivost znova. Ne predpostavljaj, da so te različice še vedno enake.

## 1.2 Obstoječe SOMA Skills

Preglej, kaj je že vgrajeno v:

- app/resources/skills/
- app/src/main/skills.ts
- app/src/main/skilllibrary.ts
- app/src/renderer/skills.ts
- app/resources/template/AGENTS.md

V pregledani različici je bilo 21 vgrajenih Skills: 3D printing, analysis, animation, assemblies, cadquery scripts, clean CadQuery modeling, CNC, design workflow, drawings, FEM, joints, laser, parametric modeling, plasma, resin/powder, sheet metal, shell environment, SOMA app, standard parts, strength in tolerances.

**Ne ustvarjaj kopij obstoječih Skills.** Najprej poišči njihove teme, ugotovi, kaj manjka, in obstoječe razširi samo tam, kjer nova pravila ali preverjeni postopki prinašajo resnično vrednost.

## 1.3 Obstoječe knjižnice in okolja

Preglej:

- python/requirements-base.txt
- python/pyproject.toml
- python/scripts/build_runtime.py
- python/soma/
- app/src/main/packages.ts
- python/soma/env.py

SOMA že ima osnovni Python runtime in ločeno projektno okolje .soma/env, v katerem se lahko namestijo dodatni paketi iz requirements.txt.

**To je pomembna prednost.** Ne dodajaj vsake knjižnice v globalni runtime. Neobvezni dodatki naj bodo nameščeni samo v projektu, v katerem se uporabljajo. V osnovni runtime dodaj samo tisto, kar je nujno za splošno delovanje SOMA.

## 1.4 Obstoječi lokalni bridge ni MCP

Preglej:

- app/src/main/bridge.ts
- app/src/main/main.ts
- app/src/main/build.ts
- python/soma/cli.py

V pregledani različici SOMA že uporablja lokalni HTTP bridge, ki posluša samo na 127.0.0.1, uporablja naključni port in žeton ter omogoča povezavo za prikaz rezultatov v aplikaciji. Trenutno je izpostavljena omejena funkcija za renderiranje.

To **ni splošen MCP strežnik**. Ne zamenjuj zasebnega bridge-a, IPC-kanalov Electron in MCP protokola.

Ne dodajaj splošnega endpointa za izvajanje poljubnega Python ali shell ukaza. To bi ustvarilo dodatno površino za izvajanje kode z uporabnikovimi pravicami.

---

# 2. Raziskane CAD-knjižnice in dodatki

Naslednji seznam je rezultat pregleda primarnih GitHub repozitorijev, README-jev, paketnih nastavitev, licenc in zadnjih commitov, kjer so bili dostopni. Oznaka »kandidat« ne pomeni, da je paket že preverjeno združljiv s SOMA 2.8.0.

## 2.1 Prva prioriteta: uradni CadQuery MCP Server

**Projekt:** [CadQuery/cadquery-contrib – mcp-server](https://github.com/CadQuery/cadquery-contrib/tree/master/mcp-server)  
**Licenca:** Apache-2.0  
**Pregledana različica paketa:** 0.2.0  
**Zahteve:** CadQuery >=2.8.0,<3, MCP Python SDK >=1.30.0,<2, Python >=3.10.  
**Zadnji pregledani commit matičnega repozitorija:** 20. 10. 2026.

Strežnik ponuja orodja:

- render – izvaja CadQuery skripto in vrne PNG ali SVG, tudi več pogledov;
- inspect – izmeri bounding box, volumen, površino, težišče in topološke podatke;
- get_parameters – prepozna parametre skripte;
- export – STEP, STL, SVG, DXF, AMF, 3MF, VRML in BREP.

Podpira sestave, imena delov, barve, lokacije in več geometrijskih objektov. Orodja imajo sheme vhodnih parametrov in preverjanje neznanih argumentov.

**Pomembno varnostno opozorilo:** uradni strežnik sprejema Python kodo kot niz in jo izvaja. Pri izvozu sprejema tudi ime ciljne datoteke. To pomeni, da ga ni varno brez omejitev izpostaviti oddaljenemu ali nezaupanja vrednemu agentu. SOMA že sama izvaja Python datoteke modelov s pravicami uporabnika, zato mora biti dodatna MCP dostopna površina še posebej dobro omejena.

**Priporočilo:** uporabi ga kot referenco za funkcionalnost in kot kandidat za lokalni test. Za končno uporabo v SOMA raje izdelaj namenski SOMA MCP adapter z omejenimi, tipiziranimi orodji. Ne izpostavljaj njegovega poljubnega code parametra neposredno uporabnikom.

## 2.2 PartCAD – najpomembnejši širši kandidat

**Projekt:** [partcad/partcad](https://github.com/partcad/partcad)  
**Licenca:** Apache-2.0  
**Pregledana različica:** 0.10.37  
**Pregledani branch:** devel  
**Zadnji pregledani commit:** 23. 11. 2026.

PartCAD je sistem za upravljanje CAD-delov in izdelkov. Uporablja deklarativno datoteko partcad.yaml in lahko upravlja:

- parametrične dele;
- sestave, podsklope in vmesnike;
- materiale in proizvodne zahteve;
- BOM/kosovnice;
- CAD-pakete in ponovno uporabo komponent;
- izdelavo dokumentacije ter montažnih navodil;
- geometrijsko preverjanje in kvalifikacijo;
- izmenjavo CAD-formatov;
- določene FEA, modalne, termične in CFD postopke;
- 2.5D CAM;
- dobavno verigo in odločitve izdelava/kupiti.

Podprti so CadQuery, build123d, OpenSCAD in drugi CAD generatorji oziroma formati, tudi STEP, BREP, IGES, STL, 3MF, OBJ, GLTF ter uvozi iz nekaterih drugih CAD sistemov.

PartCAD ima lasten MCP: pc mcp.

MCP deluje prek stdio. PartCAD uporablja lokalni daemon na ravni delovnega prostora, ki ohranja naloženi kontekst, in ločena izvajalna okolja za CAD skripte. Njegova dokumentacija opisuje izvajanje CAD wrapperjev v sandboxed Python okoljih. Podpira tudi Agent Skills ter njihovo nameščanje v različne coding agente.

**Priporočilo:** PartCAD je zelo zanimiv kot opcijska integracijska plast za knjižnice, sestave, kosovnice, materiale in deljenje CAD-komponent. Ne vgrajuj celotnega PartCAD v SOMA runtime brez dokazov o združljivosti in brez analize podvajanja funkcij. Najprej izdelaj POC z ločenim procesom in izmenjavo STEP/metadata.

**Varnost:** v pregledani dokumentaciji PartCAD izrecno piše, da je HTTP transport --http brez avtentikacije. Zato ga v integraciji ne izpostavljaj na omrežje. Uporabi lokalni stdio ali preverjeno izoliran kontejner. Ne uporabljaj zastavic, ki razširijo dovoljene poti na celoten sistem.

## 2.3 cq_warehouse – prvi kandidat za knjižnico strojnih delov

**Projekt:** [gumyr/cq_warehouse](https://github.com/gumyr/cq_warehouse)  
**Licenca:** Apache-2.0  
**Pregledana različica:** 0.8.0  
**Zadnji pregledani commit:** 16. 10. 2026.

Knjižnica CadQuery razširja s parametričnimi strojnimi deli in dodatnimi funkcijami. Njene komponente se generirajo na zahtevo in jih je mogoče izvoziti v STEP/STL ter druge formate. Zanimiva je za standardne strojne komponente, pritrdilne elemente, ležaje, zobnike in druge katalogizirane dele.

SOMA že vsebuje python/soma/fasteners.py in lastne standardne podatkovne tabele. Zato naj agent najprej primerja, kaj je v SOMA že pokrito in kaj dodatnega prinese cq_warehouse.

**Priporočilo:** prva knjižnica za izoliran integracijski test. Najprej jo namesti samo v testni projekt in preveri CadQuery 2.8.0. Ne dodajaj podvojene funkcionalnosti in ne spreminjaj globalnega runtime, dokler test ne uspe na podprtih sistemih.

## 2.4 Drugi CadQuery dodatki – selektivno, ne množično

| Projekt | Vsebina | Odločitev |
|---|---|---|
| [CadQuery/cadquery-plugins](https://github.com/CadQuery/cadquery-plugins) | Uradno sprejeti skupnostni vtičniki CadQuery | Preglej posamezne vtičnike; repozitorij ni bil zelo aktiven od 15. 8. 2024 |
| [michaelgale/cq-kit](https://github.com/michaelgale/cq-kit) | Dodatni geometrijski pripomočki, selectorji in CAD I/O | Starejši projekt; preveri vzdrževanje in združljivost, ne vgradi neposredno |
| [JustinSDK/cqMore](https://github.com/JustinSDK/cqMore) | Dodatne osnovne geometrijske metode | README temelji na CadQuery 2.1/Python 3.9; ne vključuj v osnovni sistem brez prilagoditve |
| [meadiode/cq_gears](https://github.com/meadiode/cq_gears) | Zobniki, planetna gonila, vijačni zobniki | README opozarja na nestabilnost in razvojno različico CadQuery; samo poskusni projekt |
| [cq-gridfinity](https://github.com/michaelgale/cq-gridfinity) | Gridfinity škatle, vložki in organizatorji | Opcijsko, če uporabniki dejansko potrebujejo to področje |
| [metric_threads](https://github.com/CadQuery/metric_threads) | Parametrični metrični navoji | Preveri proti obstoječim SOMA standardnim navojem in fastener knjižnici |
| [cq-electronics](https://github.com/bernhard-42/cq-electronics) | Elektronske komponente v CadQuery | Opcijsko za elektroniko |
| [KiCad Packages3D Generator](https://github.com/annem/Packages3D) | 3D modeli elektronskih komponent | Opcijsko, šele ob povezavi s KiCad |
| [ocp-freecad-cam](https://github.com/voneiden/ocp-freecad-cam) | CAM za CadQuery/build123d z uporabo FreeCAD | Ločen proces in FreeCAD; ne dodajaj v SOMA osnovni Python runtime |
| [argus-diff](https://github.com/mikelmyers/argus-diff) | Primerjava CAD geometrije in STEP/mesh regresijski testi | Raziskovalni QA kandidat; najprej preveri licenco, testno pokritost in zrelost |
| [ocp-action](https://github.com/yeicor/ocp-action) | GitHub Actions izdelava in prikaz OCP/CadQuery modelov | Uporabno kot vzorec za CI in vizualno dokumentacijo |

Dodatni katalog za nadaljnje odkrivanje: [CadQuery/awesome-cadquery](https://github.com/CadQuery/awesome-cadquery). To je seznam za raziskovanje, ne seznam paketov, ki jih smemo samodejno namestiti. Vsako kandidatno knjižnico je treba preveriti posebej.

---

# 3. Zunanji CAD/CAE sistemi in njihovi MCP strežniki

SOMA naj ostane primarno CadQuery orodje. Drugi CAD sistemi naj bodo opcijski mostovi, kadar rešujejo konkretno potrebo, ki je SOMA ne pokriva.

## 3.1 FreeCAD – najzanimivejši zunanji CAD backend

### AnkusDrive

**Projekt:** [gchen19/AnkusDrive](https://github.com/gchen19/AnkusDrive)  
**Licenca:** Apache-2.0  
**Zahteva:** FreeCAD 1.1.x  
**Zadnji pregledani commit:** 25. 11. 2026.

CLI in MCP za FreeCAD. README navaja več kot 280 strukturiranih orodij za PartDesign, skice, parametrične operacije, geometrijo, zobnike, jermene, ležaje, pritrdilne elemente, meritve, validacijo, interference in masne lastnosti, sestave, FEM, tehnične risbe, CAM, simulacije in konstrukcijske spremembe.

AnkusDrive uporablja FreeCAD-ov lasten freecadcmd in Python okolje. Ni preprosta Python knjižnica, ki bi jo lahko brez posledic namestili v SOMA runtime.

**Priporočilo:** najbolj zanimiv kandidat za namenski FreeCAD bridge, če SOMA potrebuje dodatne funkcije PartDesign, TechDraw ali določene solverje. Uporabi ga kot ločen proces z izmenjavo datotek, ne kot zamenjavo CadQuery jedra.

### TrailCurrent FreeCAD Tools

**Projekt:** [trailcurrentoss/TrailCurrentFreeCADTools](https://github.com/trailcurrentoss/TrailCurrentFreeCADTools)  
**Licenca:** MIT  
**Zahteva:** FreeCAD 0.21+, testiran na FreeCAD 1.0 in 1.1  
**Zadnji pregledani commit:** 17. 11. 2026.

Ponuja približno 100 MCP orodij za dokumente, skice, PartDesign, sestave, meritve, CAM, risbe, mesh in delo z geometrijo. Ima dodatne funkcije, kot so QR-relief, konektorske hišice in predaja izbranega telesa v Bambu Studio ali Blender.

Pomembna omejitev: orodja izvajajo operacije na FreeCAD GUI niti. Dolg klic lahko zamrzne uporabniški vmesnik, modalno pogovorno okno pa lahko blokira nadaljnje klice.

**Odločitev:** alternativa AnkusDrive, ne integriraj obeh brez jasnega razloga.

### Druge možnosti

- [sergiudanstan/freecad-mcp](https://github.com/sergiudanstan/freecad-mcp) – MIT, MCP za FreeCAD z več moduli, vključno z FEM, TechDraw, skicami in sestavi.
- [tspspi/mcpFreeCAD](https://github.com/tspspi/mcpFreeCAD) – alternativni FreeCAD MCP projekt.

Ne poveži vseh treh. Izberi enega po meritvah zanesljivosti, kompatibilnosti, orodij in stabilnosti.

## 3.2 Autodesk Fusion MCP

Autodesk ima uradni AI/MCP integraciji:

- **Fusion Local MCP** – poveže agenta z lokalno odprto aplikacijo Fusion in omogoča delo z geometrijo ter dokumentom.
- **Fusion Data MCP** – dostopa do Fusion projektov in podatkov prek Autodesk računa; lokalna aplikacija ni nujna.

Vir: [Autodesk AI Integrations](https://github.com/Autodesk/ai-integrations).

**Odločitev:** opcijska zunanja interoperabilnost za uporabnike Fusion. Zahteva Autodesk račun in ustrezno licenco. Ne dodajaj Fusion SDK v osnovni SOMA paket.

## 3.3 KiCad MCP

**Projekt:** [Seeed-Studio/kicad-mcp-server](https://github.com/Seeed-Studio/kicad-mcp-server)

Namenjen elektroniki in zahteva ustrezno različico KiCad, navedeno v projektu. Zanimiv je za pregled shem in PCB, elektronske 3D modele, ERC/DRC preverjanja in integracijo elektronskih komponent v mehanske sestave.

**Odločitev:** samo kot ločen elektronski modul, če SOMA dobi področje elektronskega/mehatroničnega načrtovanja.

## 3.4 AgentCAD in img2cad – vzorca, ne neposredna dodatka

- [n3r/AgentCAD](https://github.com/n3r/AgentCAD) – Apache-2.0. Agentic-first CAD, vendar z build123d/OpenCascade jedrom. Zanimiv vzorec za MCP, CAD validacijo, varno upravljanje Skills in delo s parametri. Ne mešaj drugega jedra v SOMA brez konkretne prednosti.
- [YUI-TIEN/img2cad](https://github.com/YUI-TIEN/img2cad) – pipeline iz slike v parametrični CAD z uporabo build123d, specifikacije, generiranja in preverjanja. Uporabi kot raziskovalni vzorec za pretvorbo skic in slik. Pred prevzemom kode preveri licenco, odvisnosti in dejansko kakovost rezultatov.

---

# 4. Skills – razširitev znanja AI agenta

## 4.1 Uporabi odprti standard Agent Skills

Specifikacija: [agentskills.io/specification](https://agentskills.io/specification)

Standardni Skill je mapa, ki vsebuje vsaj:

- SKILL.md z YAML frontmatter;
- po potrebi scripts/;
- po potrebi references/;
- po potrebi assets/.

Minimalna metapodatka sta name in description. Standard podpira postopno razkrivanje znanja: agent pozna opis Skills, navodila naloži šele ob aktivaciji, dodatne reference in skripte uporabi po potrebi.

SOMA ima že lastno obliko frontmatter, ki vključuje title, summary, order, category, default, when, topics in var. Ne zamenjaj obstoječega formata z novim brez migracijskega načrta.

Naredi kompatibilnostni sloj:

1. Sprejmi standardno mapo skill-name/SKILL.md.
2. Preveri obvezna polja, ime mape, YAML in največjo velikost.
3. Preveri licence in poreklo vseh datotek.
4. Pretvori standardni name in description v SOMA naslov in povzetek.
5. SOMA-specifična polja category, order, default, when, topics in var ustvari samo, če so smiselna in potrjena.
6. scripts/, references/ in assets/ ohrani kot ločene datoteke v Skill mapi.
7. Izvajanje tujih skript obravnavaj ločeno od navodil: namestitev Skills ne sme samodejno izvajati nepreverjene kode.
8. Vgrajeni in uporabniški Skills naj ostanejo ločeni; nadgradnja SOMA ne sme prepisati uporabniških sprememb.
9. Vzpostavi registrski zapis z izvorom, licenco, verzijo, SHA-256, združljivostjo in statusom zaupanja.
10. Dodaj testno validacijo formata in regresijo obstoječih SOMA Skills.

## 4.2 Zunanji katalogi Skills

### BaratiLab Mechanical Engineering Skills

**Projekt:** [BaratiLab/mechanical-engineering-skills](https://github.com/BaratiLab/mechanical-engineering-skills)  
**Zadnji pregledani commit:** 3. 11. 2026.

Repozitorij navaja več kot 30 inženirskih Skills, med njimi CadQuery text-to-CAD, FreeCAD query-to-CAD, parametrično izdelavo delov, sketch-to-CAD, pregled tehničnih risb in GD&T, dimenzioniranje vijakov, gredi, ležajev, vzmeti, zobnikov in zvarov, FEM, konstrukcijsko optimizacijo, 3D tisk in proizvodne postopke.

To je dobra knjižnica za izbor strokovnih postopkov. **Ne uvozi vseh Skills avtomatsko.** Preveri licenco posamezne mape, licenco skript, izvor formul, uporabljene podatke in ali je izračun mogoče neodvisno preveriti.

### PartCAD Agent Skills

PartCAD ima obsežen katalog vendor-neutral Skills v [partcad/partcad – ai-agents](https://github.com/partcad/partcad/tree/devel/ai-agents).

Primeri področij: inicializacija paketov, generiranje delov, sestavov in skic, pregled geometrije, izvoz in delo z MCP. Njihova arhitektura je zanimiva, ker iste Skills distribuira različnim agentom in jih po potrebi ponudi tudi prek MCP prompt/resource vmesnika.

**Priporočilo:** preglej in izberi postopke, ki dopolnjujejo SOMA; ne prepiši njihovega CLI workflowa, ki je specifičen za PartCAD.

### Skills.sh

[Skills.sh](https://skills.sh/) in [Vercel Skills CLI](https://github.com/vercel-labs/skills) omogočata odkrivanje in namestitev Skills iz GitHub repozitorijev.

Uporabi ga kot katalog za raziskovanje, ne kot vir, ki mu brez pregleda zaupaš. Orodje samo ne zagotavlja, da so vse Skills varne ali pravilne.

### Prednostni lastni SOMA Skills

Razvij naslednje specializirane Skills, če obstoječi ne pokrivajo dovolj:

1. CAD geometry verification – validacija BREP, volumen, bounding box, normalizacija enot, topološki pregled, geometrijski testi.
2. Photo/sketch to parametric CAD – umerjanje, perspektiva, zaznava geometrije, ločevanje opaženih mer od sklepanih mer.
3. Engineering drawing review – pogledi, prerezi, manjkajoče mere, referenčne baze, tolerance in preverjanje skladnosti risbe z modelom.
4. GD&T review – funkcionalne tolerance, baze in geometrijske tolerance; zahteva neodvisno preverljive reference.
5. Manufacturing review – CNC, laser, plazma, pločevina, varjenje, 3D-tisk, dostop orodij in minimalne debeline.
6. Parametric profile library – iztiskani profili, WPC lamele, aluminijasti profili, stebri, nosilci, konzole in spoji.
7. Assembly and interference review – sestavljanje, tolerance spojev, zračnosti, trki in preverjanje gibanja.
8. BOM and cut-list engineering – kosovnice, dolžine surovcev, odpad, kerf, standardne dolžine in revizije.
9. Material and mass properties – material, gostota, masa, površina in dokumentiranje negotovosti.
10. STEP exchange verification – izvoz in ponovni uvoz, preverjanje teles, enot, imen, barv in mer.
11. FEM setup review – obremenitve, podpore, mreža, materiali, stiki in opozorila o mejah linearnega FEM.
12. Draftwright product rules – konstrukcijska pravila, profili, nosilci in montažni postopki za ograje in terase.

---

# 5. MCP in povezovalniki

MCP je protokol za priključitev orodij k AI agentom. Ni sam po sebi varnostni sistem in ni isto kot SOMA plugin. Loči tri stvari: MCP strežnik, ki izpostavi orodja; MCP klient v coding agentu; SOMA API/bridge, ki varno izvede dovoljene operacije.

## 5.1 Predlagani SOMA MCP strežnik

Izdelaj namenski strežnik, ki je lahko ločen proces in uporablja obstoječi SOMA Python/CLI runtime.

**Prednostno uporabi stdio transport.** To je najlažji način za povezavo z lokalnimi coding agenti in ne potrebuje odprtega omrežnega porta.

Predlagana prva orodja:

### Branje in pregled
- soma_project_info
- soma_list_parts
- soma_list_parameters
- soma_get_parameter
- soma_inspect_part
- soma_measure_distance
- soma_measure_mass_properties
- soma_check_brep
- soma_list_skills
- soma_get_skill_metadata
- soma_list_exports
- soma_get_build_status

### Gradnja in spreminjanje
- soma_build
- soma_set_parameter
- soma_render
- soma_export
- soma_analyze_overhang
- soma_analyze_thickness
- soma_analyze_clearance
- soma_drawing_build
- soma_animation_build
- soma_animation_check

### Napredne operacije
- soma_fem_prepare
- soma_fem_run
- soma_fem_read_results
- soma_compare_geometry
- soma_validate_step_roundtrip
- soma_import_step
- soma_list_available_packages
- soma_install_project_package

Napredne operacije naj se dodajo šele po testih, omejitvah in jasnih opozorilih. FEM ne sme prikazovati rezultata kot certificirane inženirske potrditve.

## 5.2 Pravila za varnost MCP

To je obvezno:

- privzeto samo lokalni stdio;
- noben javni ali LAN HTTP endpoint brez izrecne potrebe, avtentikacije in varnostnega pregleda;
- ne izpostavi splošnega execute_python, eval, exec ali neomejenega shell ukaza;
- vsa orodja naj imajo stroge JSON sheme z omejenimi tipi, dolžinami, območji in enum vrednostmi;
- datoteke omeji na trenutni aktivni projekt;
- prepreči path traversal, nepooblaščene symlinke in pisanje zunaj projekta;
- izvoz dovoli samo v kontroliran ciljni imenik in dovoljene formate;
- omeji velikost vhodov, trajanje opravil, pomnilnik in število sočasnih CAD procesov;
- zunanji CAD skripti in plugin kodi ne zaupaj samo zato, ker je bila prenesena z GitHuba;
- pred brisanjem, nadpisom, premikom projekta ali namestitvijo kode zahtevaj potrditev;
- ne pošiljaj API ključev, gesel, lokalnih poti ali projektnih skrivnosti v loge;
- vsak klic zabeleži z imenom orodja, projektom, rezultatom in časom, brez razkrivanja skrivnosti;
- loči read-only orodja od spreminjajočih;
- uporabnik mora vedeti, kateri zunanji agent ima dostop do modela.

## 5.3 MCP in SOMA AI

V SOMA je prisoten notranji AI agent na osnovi Pi coding agent paketa, vendar **ne predpostavljaj, da že podpira MCP**. Preveri dokumentacijo in API točno tiste različice, ki je v app/package.json.

Če notranji agent ne podpira MCP:
- najprej izdelaj zunanji soma-mcp za Codex/Claude/Gemini/Cursor;
- nato oceni uradno podporo MCP v notranjem agentu;
- ne izvajaj nepreverjenih forkov ali hackov v notranji agent;
- ne podvajaj MCP klienta, če ga knjižnica že podpira.

## 5.4 Koristni splošni MCP strežniki in povezovalniki

To niso dodatki CAD jedru, ampak orodja, ki coding agentu pomagajo pri razvoju in preverjanju SOMA.

| Projekt | Namen | Priporočena uporaba |
|---|---|---|
| [GitHub MCP Server](https://github.com/github/github-mcp-server) | Repo, issues, PR, CI, code search | Razvoj in pregled sprememb |
| [Context7](https://github.com/upstash/context7) | Aktualna dokumentacija knjižnic | CadQuery, Electron, Python, MCP SDK in API reference |
| [Microsoft Playwright MCP](https://github.com/microsoft/playwright-mcp) | Browser avtomatizacija prek Playwright | Test SOMA spletnega mesta, docs, download strani |
| [Official MCP Servers](https://github.com/modelcontextprotocol/servers) | Filesystem, Git, Fetch, Memory in referenčni strežniki | Lokalni razvoj in raziskovanje |
| [MCP Registry](https://registry.modelcontextprotocol.io/) | Odkrivanje objavljenih MCP strežnikov | Iskanje novih CAD/CAE integracij |
| [PartCAD MCP](https://github.com/partcad/partcad) | CAD paketi, sestavi, render, kvalifikacija | Opcijski zunanji CAD ekosistem |
| [CadQuery MCP](https://github.com/CadQuery/cadquery-contrib/tree/master/mcp-server) | CadQuery render/inspect/parameters/export | POC in primer orodij |
| [FreeCAD AnkusDrive](https://github.com/gchen19/AnkusDrive) | FreeCAD CAD/CAE MCP | Opcijski zunanji FreeCAD backend |
| [KiCad MCP](https://github.com/Seeed-Studio/kicad-mcp-server) | Elektronski načrti | Samo elektronski/mehatronični moduli |
| [Autodesk AI Integrations](https://github.com/Autodesk/ai-integrations) | Fusion in podatkovni priključki | Opcijska povezava s Fusion računom |

Dodatna pravila za zunanje konektorje:

- Context7 pošilja iskalne poizvedbe svoji storitvi. Pred pošiljanjem odstrani osebne podatke, API ključe in zaupno izvorno kodo.
- Playwright omeji na dovoljene hoste in ne omogočaj poljubnega brskanja po lokalnih ali zasebnih omrežjih.
- Filesystem strežnik omeji samo na izrecno dovoljene direktorije projekta.
- Git/GitHub dostop loči med branjem in pisanjem; push, merge in brisanje naj zahtevajo potrditev.
- MCP Registry in Skills katalogi so viri za odkrivanje, ne garancija kakovosti ali varnosti.
- Ne dodajaj vseh strežnikov v privzeto konfiguracijo. Uporabnik mora aktivirati samo tiste, ki jih potrebuje.

---

# 6. Ciljna arhitektura

Priporočena arhitektura je plastna:

CODING AGENTS / SOMA AI  
Codex • Claude • Gemini • Cursor  
↓  
MCP CLIENT LAYER  
↓  
SOMA MCP + Context7/GitHub + optional PartCAD/FreeCAD  
↓  
CONTROLLED SOMA ADAPTER  
Project scope • schema validation • authentication  
↓  
SOMA CLI / SERVICES  
↓  
CAD MODEL EXECUTION ENVIRONMENT  
↓  
CadQuery 2.8 + OCP / OpenCascade  
↓  
Geometry • Assemblies • BOM • Drawings • Analysis  
↓  
STEP • STL • 3MF • DXF • PDF + validation

Dodatne knjižnice naj bodo razdeljene na:

- **core** – nujne za vsako SOMA namestitev;
- **project packages** – odvisnosti posameznega modela;
- **external bridges** – ločeni procesi za druge aplikacije;
- **skills** – navodila, reference in preverjeni skripti;
- **MCP servers** – povezovalna plast, ki kliče dovoljene funkcije.

Ne izdeluj več vzporednih CAD-jeder za isti namen. Primarni model naj ostane CadQuery/OCP, razen kadar raziskava dokaže, da je drugačen engine nujen in se uporablja kot jasno ločen backend.

---

# 7. Posebna povezava z Draftwright

**Draftwright:** [robertpezdirc12-design/draftwright-demo](https://github.com/robertpezdirc12-design/draftwright-demo)

Draftwright že pokriva fotografije, skice, umerjanje, geometrijsko prepoznavanje, specializirane profile, parametre, validacije, risbe, BOM, rezalne načrte in dokumentacijo za WoodCore/Roksal sisteme.

V pregledu njegovega README je izrecno navedena omejitev: trenutni STEP izvoz je poenostavljena box-solid aproksimacija, ne poln proizvodni CAD-model. To je pomembno področje, kjer lahko SOMA/CadQuery prinese resnično vrednost.

## Ciljni potek

1. Draftwright sprejme fotografijo, skico ali mere.
2. Draftwright izvede umerjanje in pridobi izmerjene podatke.
3. Nejasne vrednosti označi kot neznane ali ocenjene; ne izmišlja jih.
4. Izvozi strukturirano specifikacijo: profil, preseke, dolžine, kote, število elementov, položaje, material, barvo, spoje in pritrdila.
5. SOMA na osnovi te specifikacije zgradi parametričen CAD model.
6. Vsaka lamela, steber, nosilec in pritrdilni element mora biti resnična geometrija, ne samo kvader, če ima element drugačen presek.
7. SOMA preveri BREP, trke, dimenzije, spoje, maso in sestavo.
8. SOMA izdela STEP z imeni in strukturo delov, nato ga ponovno uvozi in preveri.
9. Draftwright lahko prevzame rezultat za tehnično dokumentacijo in proizvodni postopek.

## Prednostna knjižnica za Draftwright

Izdelaj parametrično knjižnico z WPC profili po dejanskih 2D presekih, aluminijastimi cevmi in profili, stebri in nosilnimi elementi, kotniki, konzolami in nosilci, pritrdilnimi elementi in vijaki, zaključnimi čepi in pokrovi, distančniki in vložki, spoji, kotnimi elementi in povezovalnimi elementi, podkonstrukcijo teras, elementi za stopnišča in povratke ter opcijskimi koriti in dodatki.

Za vsak element zabeleži:
- izvor mer in profilne risbe;
- material, gostoto in masne lastnosti;
- uporabne parametre;
- omejitve uporabe;
- način montaže;
- tolerančne zahteve;
- kako se element prikazuje v kosovnici;
- kateri test potrjuje geometrijsko pravilnost.

Ne vnesi profilov po približnih slikah. Uporabi uradne mere in preseke; manjkajoče mere označi za potrditev.

---

# 8. Izvedbeni fazni načrt

Agent naj nalogo razdeli v naslednje faze. Vsaka faza mora imeti rezultat, teste in odločitev go/no-go.

## Faza 0 – baseline in zaščita
- potrdi trenutni commit SOMA in Draftwright;
- preveri branch, status, neoddane spremembe in CI;
- zapiši Python, CadQuery, OCP, Electron in Node različice;
- ustvari ločeno razvojno vejo;
- ničesar ne briši ali prepisuj, če tega ni zahtevala naloga.

## Faza 1 – popolna inventura
- popiši vse obstoječe funkcije SOMA;
- popiši vseh 21 obstoječih Skills in njihove metapodatke;
- popiši vse osnovne in projektne Python odvisnosti;
- popiši vse CLI ukaze in njihove izhodne datoteke;
- preglej obstoječe fastener tabele in knjižnico standardnih delov;
- preglej notranji AI agent in dokazljivo podporo MCP;
- preglej SOMA bridge in njegove obstoječe route;
- naredi seznam vseh zunanjih knjižnic, ki jih že uporablja SOMA.

**Rezultat:** inventurni dokument z datotekami, funkcijami, odvisnostmi in manjkajočimi funkcijami.

## Faza 2 – CAD dodatek POC
- namesti cq_warehouse v izolirano testno okolje z obstoječim SOMA CadQuery;
- preizkusi izbrane komponente in primerjaj z SOMA fasteners;
- izmeri geometrijo, volumen, težišče, bounding box in topologijo;
- izvoz STEP, ponovni uvoz STEP in preverjanje;
- test Windows, Linux in macOS;
- ne spreminjaj osnovnega runtime, če ni potrebe.

**Rezultat:** tabela kompatibilnosti, funkcij, stroškov, napak in odločitve.

## Faza 3 – SOMA MCP POC
- najprej implementiraj read-only orodja;
- poveži jih z obstoječim CLI/service modelom;
- uvedi stroge sheme in omejitve projekta;
- uporabi stdio;
- preizkusi Codex in vsaj en drug MCP klient;
- preveri, da MCP klic ne more dostopati do datotek zunaj projekta;
- preveri, da poljubna koda ni izpostavljena kot splošno orodje.

**Rezultat:** delujoč lokalni SOMA MCP z dokazili testov.

## Faza 4 – Skills standard in katalog
- zgradi compatibility importer za standard Agent Skills;
- shrani izvor, licenco, verzijo in hash;
- uvedi zaupanje: trusted, review-required, blocked;
- dodaj prve izbrane CAD/engineering Skills;
- dodaj validacijo Skills in regresijske teste;
- omogoči, da se uporabniške spremembe ohranijo.

**Rezultat:** nadzorovan katalog Skills, ne nenadzorovan kopirni sistem.

## Faza 5 – PartCAD POC
- preizkusi pc mcp prek stdio;
- preizkusi uvoz in izvoz STEP med SOMA in PartCAD;
- primerjaj upravljanje delov, sestavov, BOM, materialov in package cache;
- preizkusi izolacijo CAD skript in omejitev poti;
- preveri vpliv na velikost namestitve in čas zagona;
- odloči se, ali PartCAD ostane zunanji bridge ali dobi namenski adapter.

**Rezultat:** poročilo, ali PartCAD prinaša dovolj koristi, da ga je vredno podpirati.

## Faza 6 – geometrijski QA
- dodaj BREP validity test;
- dodaj STEP round-trip test;
- preveri enote in koordinate;
- preveri sestavo in imena delov;
- preveri volume/mass delta;
- preveri interference in zračnosti;
- dodaj štiri poglede (front, top, side, isometric);
- preveri, da izrisana risba ustreza 3D modelu in parametrom;
- raziskuj argus-diff kot izbirni CI test.

**Rezultat:** ponovljiv geometrijski QA sistem.

## Faza 7 – Draftwright CAD knjižnica
- pripravi specifikacijski format med Draftwright in SOMA;
- implementiraj realne profile in podsestave;
- izdela model vsakega elementa s parametri;
- dodaj BOM in risbe;
- izvoz STEP/3MF/STL;
- pripravi testne projekte, ki imajo znane referenčne mere.

**Rezultat:** dokaz, da SOMA izdeluje realne konstrukcijske modele iz Draftwright specifikacij.

## Faza 8 – dokumentacija in distribucija
- posodobi SOMA vodič;
- dokumentiraj namestitev, izključitev in odstranitev MCP strežnikov;
- dokumentiraj licence in tretje odvisnosti;
- dodaj primere MCP konfiguracije za podprte agente;
- posodobi showcase spletnega mesta šele, ko so primeri dejansko zgrajeni in preverjeni;
- ne objavi funkcije, ki je samo prototip ali še nima opravljenega testa.

---

# 9. Obvezni specializirani podagenti

Razdeli delo vzporedno, kadar je smiselno. Vsak podagent mora dobiti natančen predmet pregleda in pričakovani izdelek:

| Specialist | Kaj mora preveriti/izdelati |
|---|---|
| Software Architect | Arhitektura SOMA, meje med Electron/Python/CAD/MCP, verzioniranje API |
| CAD/CadQuery specialist | Geometrijske knjižnice, parametrični deli, assembly, STEP |
| Python packaging specialist | Python 3.12, CadQuery 2.8, OCP, wheels, izolirana okolja |
| MCP specialist | SDK, transporti, sheme, klienti, resources, prompts, orodja |
| Security specialist | arbitrary code, filesystem, subprocess, tokeni, kontejner, path traversal |
| Skills specialist | Standard Agent Skills, SOMA frontmatter, importer, trust in licenca |
| Licensing auditor | GPL, Apache-2.0, MIT, LGPL, third-party notices in redistribucija |
| QA specialist | unit, integration, E2E, geometry regression, STEP round-trip |
| Mechanical engineer | Profili, spoji, materiali, tolerance, proizvodljivost |
| CAE/FEM specialist | Mreže, pogoji, solverji, numerična validacija in meje analiz |
| Documentation specialist | SOMA vodič, MCP konfiguracija, primeri in navodila uporabnikom |

Vsak specialist mora navesti, kaj je dejansko pregledal, točne poti in reference, potrjene ugotovitve, dvome in tveganja, predlog sprememb ter teste, ki dokazujejo ugotovitev.

Koordinator mora rezultate združiti, odpraviti podvajanja in izpostaviti protislovja. Ne sprejmi splošnih priporočil brez pregleda kode.

---

# 10. Licenciranje in distribucija

SOMA je GPL-3.0-or-later z ločeno SOMA Model Exception. Izjema omogoča uporabniku poljubno licenciranje njegovih Model files, Project Files in Output, vendar ne spremeni licence same aplikacije SOMA.

Pred vključitvijo vsake zunanje knjižnice:
1. preveri primarno LICENSE datoteko;
2. preveri licence vseh odvisnosti;
3. preveri, ali licenca dovoljuje distribucijo z SOMA;
4. preveri, ali je treba dodati NOTICE/copyright;
5. ohrani izvor in različico;
6. posodobi THIRD_PARTY_NOTICES.md;
7. ne kopiraj kode, če licenca ali izvor ni jasen.

Posebej preveri Apache-2.0, MIT, BSD, LGPL in GPL odvisnosti. Ne predpostavljaj, da je licenca glavnega GitHub repozitorija enaka licenci vseh priloženih datotek, modelov, podatkov ali skript.

Ne spreminjaj licence SOMA.

---

# 11. Testna in sprejemna merila

Razširitev se šteje kot zaključena šele, ko uspe naslednje:

### Namestitev
- uspešna namestitev na Windows 10/11;
- uspešna namestitev na podprtem Linuxu;
- macOS, če ga SOMA uradno podpira;
- zaklenjene verzije odvisnosti;
- preverjanje, da se osnovni runtime ne poškoduje;
- projekt lahko uporablja lastne pakete brez posega v sistemsko Python namestitev.

### CAD
- model se zgradi brez napak;
- vsak pričakovani element je registriran;
- vsak BREP je veljaven;
- dimenzije se ujemajo z izhodiščnimi parametri;
- volumen in masa sta skladna z referenčnimi izračuni;
- sestavi ohranijo imena, lokacije, barve in hierarhijo;
- interference testi ne pokažejo nepričakovanih trkov.

### Izvoz
- STEP se izvozi;
- STEP se ponovno uvozi;
- število teles in sestavna hierarhija se ujemata;
- dimenzije, volumen in enote se ne spremenijo preko določene tolerance;
- STL/3MF so zaprti in primerni za mesh workflow;
- DXF/PDF risba ustreza dejanskemu modelu.

### MCP
- initialize in tools/list delujeta;
- vsako orodje ima test veljavnih in neveljavnih vhodov;
- napačen projekt ali pot je zavrnjena;
- MCP ne more prebrati zunanjih datotek;
- ni dostopa do proizvoljnega shell ukaza;
- nepooblaščen klic ne more izvesti spreminjajoče operacije;
- dolg CAD ukaz se lahko prekine ali časovno omeji;
- po napaki strežnik ostane odziven.

### Skills
- validacija metapodatkov;
- zavrnitev neveljavnega YAML;
- opozorilo o nezaupanja vrednem izvoru;
- ohranitev uporabniških sprememb;
- postopno nalaganje referenc;
- nobena tuja skripta se ne izvede samo z namestitvijo Skill.

### FEM/CAE
- primerjaj rezultate z analitičnimi referenčnimi primeri;
- naredi mesh convergence;
- dokumentiraj materialne predpostavke;
- preveri ravnotežje sil in reakcij;
- jasno prikaži omejitve linearnega modela;
- ne oglašuj FEM kot certificiranega izračuna brez neodvisne validacije.

---

# 12. Pravila za delo na repozitorijih

- SOMA koda se spreminja samo v [robertpezdirc8-prog/SOMA](https://github.com/robertpezdirc8-prog/SOMA).
- Ta repozitorij soma-website je namenjen dokumentaciji, vodičem, showcase in objavi informacij.
- Draftwright ostane ločen projekt; integracija naj uporablja dokumentiran format specifikacije, ne kopiranja celotne aplikacije.
- Pred spremembami preveri neoddane lokalne spremembe in delo drugih agentov.
- Ne prepiši ali odstrani tujih neoddajnih sprememb.
- Delaj v novi veji.
- Ne pushaj neposredno na main.
- Ne sproži produkcijske izdaje ali migracije brez izrecne zahteve.
- Ne spreminjaj neodvisnih datotek, ki niso del potrjenega načrta.
- Vsaka zaključena faza mora imeti commit, povzetek in teste.
- Če zunanja knjižnica ne deluje, dokumentiraj razlog in jo izloči; ne prilagajaj SOMA na silo samo zato, da bi jo vključil.

---

# 13. Končno poročilo, ki ga mora agent vrniti

Na koncu pripravi pregledno poročilo s temi poglavji:

1. **Kaj je SOMA že imela** – prepreči podvajanje.
2. **Kaj je bilo raziskano** – seznam repozitorijev, verzij, licenc in aktivnosti.
3. **Kaj je dejansko dodano** – datoteke, funkcije, odvisnosti in UI.
4. **Kaj je ostalo kot zunanji bridge** – razlogi in namestitev.
5. **Kaj je zavrnjeno** – nezdružljivost, stara koda, licenca, varnost ali premalo vrednosti.
6. **Skills** – nove, prilagojene in obstoječe; vir, licenca, hash.
7. **MCP** – transport, seznam orodij, sheme, varnostne omejitve, preizkušeni klienti.
8. **Draftwright povezava** – specifikacija, primer in rezultat.
9. **Testi** – ukazi, okolja, rezultati, napake in nerešene težave.
10. **Licenčna dokumentacija** – vse spremembe NOTICE/THIRD_PARTY_NOTICES.
11. **Omejitve** – kaj še ni pripravljeno za produkcijo.
12. **Priporočeni naslednji korak** – en jasen prioritetni predlog.

Pri vsaki trditvi »deluje« priloži dokaz: test, izhod, screenshot, validiran model ali uspešen round-trip. Če ni dokazano, označi kot »POC«, »eksperimentalno« ali »še ne preverjeno«.

---

# 14. Končna prioriteta

Ne poskušaj narediti vsega hkrati. Priporočeni vrstni red:

1. **SOMA inventory + baseline**
2. **CadQuery 2.8 MCP in namenski SOMA typed MCP**
3. **cq_warehouse v izoliranem projektu**
4. **Agent Skills standard + varni importer**
5. **PartCAD POC za pakete, sestave in BOM**
6. **Geometry verification + STEP round-trip + CI**
7. **Draftwright parametrična knjižnica profilov in spojev**
8. **FreeCAD/AnkusDrive bridge samo za funkcije, ki jih SOMA res nima**
9. **Dodatni CAM, KiCad, Fusion in drugi konektorji samo za dejanske primere uporabe**
10. **Dokumentacija, vodiči in showcase šele po uspešnem preverjanju**

**Končni cilj:** SOMA naj ne bo samo CAD program z velikim številom dodatkov. Postati mora varno razširljiv CAD sistem, v katerem AI agenti uporabljajo preverjene geometrijske funkcije, strokovne Skills in zunanje povezovalnike, medtem ko so modeli, meritve, izvozi in konstrukcijske odločitve sledljivi ter preverljivi.
