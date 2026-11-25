# SOTU — vinyl player per Spotify

Web app personale (non commerciale) che mostra la musica Spotify come vinili: cassa di dischi in Cover Flow verticale → il disco va sul giradischi. Tributo al negozio **Sounds of the Universe** di Londra (con la S finale). È per il figlio dell'utente (Spotify Family, Premium).

## Chi è l'utente
- Grafico, ama tipografia e minimalismo. **Non è uno sviluppatore**: spiegazioni tecniche semplici, passo passo, in **italiano**.
- Orari sempre in fuso italiano (CET/CEST).
- Vuole coerenza fisica e prospettica: niente geometrie "inventate". Quando manda riferimenti intende il comportamento/prospettiva, non i dettagli letterali.
- Token limitati: lavorare in modo mirato, niente esplorazioni lunghe del file.

## File
- `index.html` — tutta l'app (HTML + CSS + JS vanilla, ~1050 righe). Nessun build.
- `arm-base.svg`, `arm-move.svg` — braccio del giradischi (dal suo bacchetto.svg). Pivot nel viewBox 100×75.2 a (79.2, 19.4); `ARM = {px:79.2, py:19.4, tip:54.04, a0:26.9}`. Riferiti con `?v=4` (cache Safari).
- `sotu-logo.svg` — logo dell'utente (viewBox ritagliato al contenuto per allineamento a sinistra perfetto). Usato in accesso e in cima all'elenco playlist, con `?v=2`.
- `manifest.webmanifest`, `apple-touch-icon.png`, `icon-192/512.png` — web app su Home iPhone.

## Struttura dell'app (viste: `gate`, `lists`, `crate`, `player`)
- **Accesso** (`gate`): PKCE OAuth nel browser; redirect = `location.origin + location.pathname`. Demo locale con `?demo=1`.
- **Elenco** (`lists`): logo, "Libreria" (Brani che ti piacciono / Album salvati), "Playlist" (solo le proprie: in Development Mode `/playlists/{id}/items` funziona solo per playlist proprie/collaborative; il campo è `item`).
- **Cassa** (`crate`): Cover Flow verticale, i brani della playlist raggruppati per album (`groupByAlbum`) così si ascolta l'album intero. `layoutCrate` / `sleeveTransform`: copertina centrale a 0°, laterali `rotateX(±72°)`, fitte. La prospettiva è sul `.crate` (genitore diretto delle copertine).
- **Tocco sul disco centrale** → `goPlay(i)`: la copertina si alza, il vinile esce **dall'alto** della busta durante lo zoom, esce tutto, poi va sul piatto; parte dal primo brano della playlist in quell'album (`startIdx`). Niente più schermata di rotazione (le funzioni `pick/flipLifted/putBack/toTurntable` restano ma non sono più il percorso principale).
- **Giradischi** (`player`): piatto + vinile (colore dalla copertina, `vinylColor`), etichetta = copertina, perno sul piatto (non sul disco), braccio SVG (`armAngleFor`, `updateArm`), lucina rossa (`#lamp`, `#glow` sopra il disco). Tasti tondi trasparenti con filetto grigio, direttamente sul piano; tasto **espelli** sopra. Sotto: copertina grande che sborda in basso; swipe orizzontale (gestito a mano con pointer events) → tracklist sul fondo nero.
- **Espelli / tocco copertina** → `stowRecord`: pausa, il vinile rientra dall'alto nella busta, la copertina torna nella cassa (contrario dell'andata).
- Animazioni: `path()` (tappe con easing morbido) e `glide()`; misure prese prima di partire, cassa preparata con `quietCrate`.

## Scelte di design già decise (non tornare indietro senza chiedere)
- Sfondo **nero** ovunque (niente legno). Testi **Helvetica Neue**, bianchi, maiuscole normali, 17px per la lettura; niente display verdi.
- Tracklist senza fondi grigi/beige: testo direttamente sul nero.
- Tasti piatti (niente 3D). Freccia indietro stile iOS (chevron sottile).
- La cassa **non deve mai essere semitrasparente** (Safari appiattisce il 3D): sta sotto con `z-index:1`, sfuma la vista sopra di lei.
- Avvisi (`#toast`) nascosti con opacity/visibility (non solo translate).

## Riproduzione
- Web Playback SDK solo su desktop (non su browser mobile → `MOBILE`). Zen/Firefox: audio si blocca, Safari ok.
- `pickDevice`: prima app Spotify attiva, poi SDK, poi altri dispositivi. `playAlbum` toglie lo shuffle prima e dopo il play.
- `api()` riprova da sola sugli errori 5xx (Bad gateway). Se l'app Spotify su iPhone "dorme" (sparisce dai dispositivi): avviso con bottone **Apri Spotify** (`spotify:`), al ritorno (`visibilitychange`) il disco riparte da solo (`pendingPlay`).

## Pubblicare (deploy)
1. Modifiche in `/home/claude/sotu` (repo `hellosandro/sotu`, branch `main`). Commit con i trailer richiesti dalla sessione, poi `git push`.
2. Vercel MCP `create_deployment` con `teamId: team_znRhvS5Yo2uOek6I31MCsAk6`, `requestBody: {name:"sotu", project:"sotu", target:"production", gitSource:{type:"github", org:"hellosandro", repo:"sotu", ref:"main", sha:<commit>}}`.
3. Controllare con `get_deployment` finché `READY`. Produzione: **sotu-eight.vercel.app**.

## Provare in locale
- Copiare i file in `/tmp/vinile/` e aprire `file:///tmp/vinile/index.html?demo=1` con Playwright (Chrome `/usr/bin/google-chrome`, `--no-sandbox`), viewport iPhone 390×844 dpr 2; Mac 1100×1000 (colonna app 480px al centro).
- Chrome non riproduce i bug di Safari (3D appiattito): chiedere all'utente di verificare su Safari/iPhone.

## Problemi noti
- iOS 26.x: nella web app da Home restano bande nere sopra e sotto (bug WebKit 301994), non risolvibile dalla pagina.
- Spotify su iPhone si addormenta quando resta in pausa in secondo piano: l'utente ha scelto di lasciare la pausa vera.

## Da fare (lista dell'utente)
1. ~~Creare CLAUDE.md~~ (fatto)
2. Risolvere il problema dell'ascolto Spotify in pausa (idea proposta: brano di silenzio al posto della pausa per tenere Spotify sveglia).
3. Chiusura del disco: tornando al Cover Flow a un certo punto si vede "sbucare" il disco.
4. Versione orizzontale o blocco in verticale. In orizzontale: Cover Flow identico ma solo le copertine ruotate di 90°; ascolto con giradischi a sinistra e album a destra; elenco playlist: sfruttare meglio lo spazio o lasciare lo scroll verticale.
5. Pulsazione leggera della lucina rossa per renderla viva.
