## restart lichess

`systemctl restart lichess`

To be used only if the majority of players is affected. Restarting lichess is especially bad at peak hours. It can take a while when 4000+ players are connected.


## reading debug logs

`tail -20 lichessjournal`  
Replace 20 with the line count to display desired. The log file is sometimes very large (`wc -l lichessjournal`) so it's probably better to only print the last bulk of it.

## access database

`mongo` to start up the REPL environment  
`> use lichess`  

From there it is possible to query collections. For example,  
`> db.user4.find().length()`

Be extremely careful. To be used only for damage control. Some queries can be slow enough to overload the server (like querying the 130 million games without a proper index).

## terminate tournament

__NOTE: ONLY FOR EXTREME MEASURES!__  
In the db: `lichess> db.tournament2.update({_id: "tournamentID"}, {$set: {status: NumberInt(30), winner: "winnerID"}})`

## people with server access

- thibault on IRC / or text/email thibault.duplessis@gmail.com / LINE (id: thibalest)
- Unihedron on IRC / or IM via LINE (id: unihedron) or Whatsapp (id: contact) or text vincentyification@gmail.com
- flugsio on IRC