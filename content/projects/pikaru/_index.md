+++
title = "Pikaru - Pick a room to study in"
+++

At the _Politecnico Leonardo_ campus, it was often hard to find some
place to study in. _Politecnico_ does provide a 
[website](https://www7.ceda.polimi.it/spazi/spazi/controller/RicercaAuleLibere.do)
that lists free classrooms based on specific dates and time slots, 
for which a former student implemented a
[Telegram bot](https://t.me/auleliberepolimi_bot), making the whole
process simpler.
However, these tools still make it hard to plan for a whole day of 
study, since rooms are often not free all day.

This project then aims to **implement an algorithm and interface that
optimises a full day study session**, choosing rooms to stay in based on their occupation and the student's schedule, while **minimising 
displacement** between rooms. Ideally, it'd also allow the student to
set a preference for specific rooms, or rooms with specific features
(e.g. plugs).

This project was inspired by the use of Integer Linear Programming to 
optimise classroom allocation in [USPolis](https://uspolis.com.br/).

---
