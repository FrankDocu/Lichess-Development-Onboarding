When a user sends an email about not receiving a confirmation email, here's the workflow:

1. Copy their email address from gmail, and search for it on https://lichess.org/player
2. If the user is found:
  1. See if they have a game played.
  2. If the user has a game, it means they could eventually log in. Discard their request, we're done.
  3. If no game, open the user mod view, and click the tick to confirm their email. Then use the "Email confirmed" canned response. Done.
3. If the user is not found:
  1. See if they mentioned their username in their request email.
  2. If they mentioned their username:
    1. Search for it on https://lichess.org/player
    2. If the user is found, open the mod view, set their email (from their request email) and click the tick. Send the canned response "Email: Confirmed". Done.
    3. If not, tell them they didn't register, either with that email or that username. Done.
  3. If they did not mention their username:
    1. Ask for it with the canned response "Misc: Username?". Done.