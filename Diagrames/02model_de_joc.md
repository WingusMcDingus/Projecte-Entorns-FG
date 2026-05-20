## 1. Resum del joc

Joc de lluita 1v1 on el jugador s’enfronta a una IA. Cada personatge té 10 punts de vida. Les rondes es guanyen quan l’oponent arriba a 0 vida. Guanya la partida el primer que aconsegueix 2 rondes.

---

## 2. Explicació del diagrama de classes

El diagrama de classes mostra les entitats principals del joc i com es relacionen.

- **GameManager**: controla l’estat global del joc, les rondes i el flux del partit.  
- **JugadorHumà**: llegeix l’input del jugador i controla el seu personatge.  
- **JugadorIA**: la IA tria accions i controla el seu personatge.  
- **Personatge**: té 10 punts de vida, un estat, direcció i una llista de 4 atacs.  
- **Atac**: defineix els 4 tipus d’atac (Low, Overhead, Light, Heavy/Combo). Cada atac fa 1 punt de dany excepte el Heavy/Combo que en fa 2.  
- **Enums**: defineixen estats del joc, direccions, estats del personatge i tipus d’atac.

### Relacions principals

- GameManager compon els dos jugadors (1 a 1).
- Cada jugador controla un Personatge (1 a 1).
- Cada Personatge té exactament 4 Atacs (1 a 4).

---

## 3. Explicació del diagrama d’activitats

Representa el flux del joc i el bucle principal durant una partida.

1. S’inicialitza el joc (vida a 10 i rondes a 0).
2. Selecció de personatges: el jugador tria, la IA escull random.
3. Comença una ronda i es posicionen els personatges.
4. Bucle de combat (executat cada frame):
   - Es llegeix l’input del jugador i la IA decideix acció.
   - Es mouen els personatges.
   - S’executen atacs, bloquejos i es detecten col·lisions.
   - S’aplica el dany i s’actualitza la UI.
5. Si algú queda K.O., acaba la ronda i s’actualitzen les rondes guanyades.
6. Si ningú té 2 rondes, s’inicia una nova ronda.
7. Si algú té 2 rondes, acaba la partida i es mostra el guanyador.

---

## 4. Notes de disseny importants

- Moviment universal: caminar endavant/endarrere, saltar, agachar i bloquejar. Dash = dos cops endavant ràpids.
- Cada personatge té els mateixos 4 tipus d’atac, però amb animacions diferents.
- Els atacs fan 1 punt de dany, excepte Heavy/Combo que en fa 2.
- Les rondes són independents, la vida es reinicia cada ronda.