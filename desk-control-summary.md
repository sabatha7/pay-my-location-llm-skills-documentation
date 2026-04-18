User request
 → Identify intents

 → More than one?
     → No → continue
     → Yes:
         → Are they related?
             → Yes → compose into parent intent
             → No → split into tickets (queue)

 → For each intent (or composed flow):
     → Load requirements
     → Check arguments
         → Missing → ask
         → Complete → execute
