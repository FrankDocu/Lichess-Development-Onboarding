## restart lichess

`systemctl restart lichess`

## when hooks pile up, slowing down the server

-> restart lichess

## reading debug logs

`tail -20 lichessjournal`  
Replace 20 with the line count to display desired. The log file is sometimes very large (`wc -l lichessjournal`) so it's probably better to only print the last bulk of it.

## access database

`mongo` to start up the REPL environment  
`> use lichess`  

From there it is possible to query collections. For example,  
`> db.user4.find().length()`

## terminate tournament

__NOTE: ONLY FOR EXTREME MEASURES!__  
In the db: `lichess> db.tournament2.update({_id: "tournamentID"}, {status: NumberInt(30), winner: "winnerID"})`

## fix broken modzones

## people with server access

- thibault on IRC / or text thibault.duplessis@gmail.com / LINE (id: thibalest)
- Unihedron on IRC / or IM via LINE (id: unihedron) or Whatsapp (id: contact) or text vincentyification@gmail.com
- flugsio on IRC