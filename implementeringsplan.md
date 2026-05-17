# Implementeringsplan for MatSpil

## Formål
- Gøre spillet sværere på en fair måde.
- Vise reducerede svar i højre side som heltal + brøk.
- Belønne hurtighed med progressiv bonus uden straf.
- Understøtte uægte brøker (tæller større end nævner).

## Overordnede principper
- Ingen minuspoint for langsom tid.
- Hurtig løsning giver stigende bonus.
- Dynamiske tidsmål beregnes pr. spiller ud fra tidligere runder.
- Match-logik bliver ID-baseret, så UI-format ikke bryder spilregler.

## Fase 1: Visning af blandede tal i højre side
1. Tilføj formattering af reduceret brøk til blandet tal:
   - h = floor(n/d)
   - r = n mod d
2. Visningsregler:
   - Hvis resten r er 0: vis kun heltal (fx 3).
   - Hvis heltalsdelen h er 0: vis brøk (fx 2/7).
   - Ellers: vis blandet tal (fx 3 2/5).
3. Sørg for at korrekt/forkert stadig afgøres af kortenes interne ID.

## Fase 2: Stigende sværhedsgrad
1. Indfør en intern difficultyScore (1-10).
2. Opdater score efter hver runde ud fra:
   - Hastighed (ift. tidsmål).
   - Fejlrate.
   - Eventuel streak af gode runder.
3. Map difficultyScore til niveauer:
   - 1-3: Let
   - 4-6: Mellem
   - 7-10: Svær
4. Lad disse parametre stige med difficultyScore:
   - Antal par pr. runde.
   - Talstørrelser.
   - Faktorinterval ved udvidelse af brøker.
   - Andel uægte brøker.

## Fase 3: Uægte brøker
1. Let: 0 procent uægte brøker.
2. Mellem: cirka 30 procent uægte brøker.
3. Svær: cirka 60 procent uægte brøker.
4. Sørg for gyldige data:
   - Nævner er større end 0.
   - Tæller er ikke det samme som nævner.
   - Reduceret form beregnes korrekt med gcd

## Fase 4: Dynamisk tid og progressiv bonus
1. Gem pr. spiller:
   - targetTimeSec (aktuelt tidsmål)
   - timingBaseline (løbende forventning)
2. Opdater baseline efter hver runde med glatning (EWMA):
   - ny = 0.75 * gammel + 0.25 * rundetid
3. Beregn næste tidsmål ud fra baseline med let progressiv stramning.
4. Bonusregel uden straf:
   - Hvis rundetid er mindre end eller lig målet: giv bonus med progressiv kurve.
   - Jo hurtigere i forhold til mål, jo højere bonus.
   - Hvis rundetid er større end målet: bonus er 0 (ikke negativ).
5. Sæt loft på bonusprocent for balance.

## Fase 5: Scoremodel
1. Basepoint pr. korrekt match (fx 10).
2. Runde-basepoint = korrekt antal * basepoint.
3. Hastighedsbonus lægges oveni.
4. Gem scorefelter:
   - dato
   - runder
   - basePoints
   - speedBonus
   - totalScore

## Fase 6: UI-feedback
1. Efter hver runde vises:
   - Rundetid
   - Tidsmål
   - Basepoint
   - Hastighedsbonus
   - Ny total score
2. Brug positiv feedback til nudging:
   - Hurtigere tid udløser tydelig visuel belønning.
   - Ingen røde straf-beskeder for langsom tid.

## Fase 7: Bonusspørgsmål om matematiske termer
1. Efter hver runde vises 1 bonusspørgsmål.
2. Spørgsmålstype:
   - Multiple choice med 3 svarmuligheder.
3. Point:
   - Korrekt svar giver +5 bonuspoint.
   - Forkert svar giver 0 point.
   - Ingen straf.
4. Tidsregel:
   - Bonusspørgsmål tæller ikke med i rundens tidsmåling.
5. Variation:
   - Samme spørgsmål må ikke komme to runder i træk.

## Visualisering: Flow efter runde

1. Runde færdig
2. Runde-resultat vises
3. Bonusspørgsmål modal vises
4. Svar gives
5. Bonuspoint tilføjes
6. Næste runde

```text
[Runde færdig]
      |
      v
[Resultatpanel]
  tid, mål, basepoint
      |
      v
[Bonusspørgsmål]
  +5 ved korrekt
      |
      v
[Næste runde]
```

## Visualisering: Eksempel på bonusspørgsmål

```text
+--------------------------------------------------+
| Bonusspørgsmål                                  |
| Hvad kaldes tallet øverst i en brøk?            |
|                                                 |
|  A) Nævner                                      |
|  B) Tæller                                      |
|  C) Faktor                                      |
|                                                 |
| [Svar]                                          |
+--------------------------------------------------+
```

## Visualisering: Term med eksempel

```text
      3
     ---
      4

Tæller = 3 (øverst)
Nævner = 4 (nederst)
```

Forslag til term-spørgsmål:
- Hvad betyder tæller?
- Hvad betyder nævner?
- Hvad er forkortning af en brøk?
- Hvad er en uægte brøk?
- Hvad er største fælles divisor?

## Testplan
1. Verificer visning af blandede tal:
   - 17/5 bliver til 3 2/5
   - 9/3 bliver til 3
   - 2/7 bliver til 2/7
2. Verificer at match-logik stadig er korrekt.
3. Spil hurtige og langsomme runder:
   - Hurtige runder giver stigende bonus.
   - Langsomme runder giver mindre eller ingen bonus, aldrig straf.
4. Skift spiller og bekræft separat dynamisk tidsmodel.
5. Kontroller at scorehistorik gemmer alle nye felter.
6. Verificer bonusspørgsmål:
   - Der vises præcis 1 bonusspørgsmål efter hver runde.
   - Korrekt svar giver +5, forkert giver 0.
   - Bonusspørgsmål påvirker ikke rundens tidsbonus.
   - Samme spørgsmål gentages ikke to runder i træk.

## Done-kriterier
- Højresiden viser heltal + brøk korrekt.
- Uægte brøker optræder i mellem og svær.
- Sværhedsgrad stiger adaptivt over runder.
- Dynamisk tidsbonus virker uden straf.
- Resultater gemmes med dato og bonus-opdeling.
- Bonusspørgsmål om matematiske termer vises efter hver runde.
