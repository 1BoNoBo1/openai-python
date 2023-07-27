(Ce document est un aperçu du format sous-jacent utilisé par
Modèles ChatGPT. En tant que développeur, vous pouvez utiliser notre [niveau supérieur
API](https://platform.openai.com/docs/guides/chat) et n'aura pas besoin de
interagir directement avec ce format aujourd'hui, mais attendez-vous à avoir le
option à l'avenir !)

Traditionnellement, les modèles GPT consommaient du texte non structuré. Modèles ChatGPT
attendez-vous plutôt à un format structuré, appelé Chat Markup Language
(ChatML en abrégé).
Les documents ChatML consistent en une séquence de messages. Chaque message
contient un en-tête (qui consiste aujourd'hui en celui qui l'a dit, mais dans le
contiendra d'autres métadonnées) et des contenus (qui aujourd'hui sont
charge utile de texte, mais contiendra à l'avenir d'autres types de données).
Nous continuons à faire évoluer ChatML, mais la version actuelle (ChatML v0) peut
être représenté avec notre prochain format JSON "liste de dicts" comme
suit :
```
[
 {"token": "<|im_start|>"},
 "system\nYou are ChatGPT, a large language model trained by OpenAI. Answer as concisely as possible.\nKnowledge cutoff: 2021-09-01\nCurrent date: 2023-03-01",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "user\nHow are you",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "assistant\nI am doing well!",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "user\nHow are you now?",
 {"token": "<|im_end|>"}, "\n"
]
```
Vous pouvez également le représenter dans la classique "chaîne brute non sécurisée"
format. Cependant, ce format permet intrinsèquement des injections de l'utilisateur
entrée contenant une syntaxe de jeton spécial, similaire aux injections SQL :
```
<|im_start|>system
You are ChatGPT, a large language model trained by OpenAI. Answer as concisely as possible.
Knowledge cutoff: 2021-09-01
Current date: 2023-03-01<|im_end|>
<|im_start|>user
How are you<|im_end|>
<|im_start|>assistant
I am doing well!<|im_end|>
<|im_start|>user
How are you now?<|im_end|>
```
## Cas d'utilisation hors chat
ChatML peut être appliqué aux cas d'utilisation GPT classiques qui ne sont pas
traditionnellement considéré comme un chat. Par exemple, l'instruction suivant
(lorsqu'un utilisateur demande à l'IA de terminer une instruction) peut être
implémenté en tant que requête ChatML comme suit :
```
[
 {"token": "<|im_start|>"},
 "user\nList off some good ideas:",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "assistant"
]
```
Nous n'autorisons pas actuellement la saisie semi-automatique de messages partiels,```
[
 {"token": "<|im_start|>"},
 "system\nPlease autocomplete the user's message.",
 {"token": "<|im_end|>"}, "\n", {"token": "<|im_start|>"},
 "user\nThis morning I decided to eat a giant"
]
```
Notez que ChatML rend explicite au modèle la source de chaque pièce
du texte, et montre en particulier la frontière entre l'humain et l'IA
texte. Cela donne l'occasion d'atténuer et éventuellement de résoudre
injections, car le modèle peut dire quelles instructions proviennent du
développeur, l'utilisateur ou sa propre contribution.
## Invite à quelques prises de vue
En général, nous vous recommandons d'ajouter des exemples de quelques plans en utilisant des
messages `system` avec un champ `name` de `example_user` ou
`exemple_assistant`. Par exemple, voici une invite à un coup :
```
<|im_start|>system
Translate from English to French
<|im_end|>
<|im_start|>system name=example_user
How are you?
<|im_end|>
<|im_start|>system name=example_assistant
Comment allez-vous?
<|im_end|>
<|im_start|>user
{{user input here}}<|im_end|>
```
Si l'ajout d'instructions dans le message `system` ne fonctionne pas, vous pouvez
essayez également de les mettre dans un message "utilisateur". (Dans un futur proche, nous
entraînera nos modèles à être beaucoup plus orientables via le système
message. Mais à ce jour, nous ne nous sommes entraînés que sur quelques messages système,
les modèles accordent donc beaucoup plus d'attention aux exemples d'utilisateurs.)