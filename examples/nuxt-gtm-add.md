Ajouter GTM sur le frontend du projet dans `front/nuxt.config.ts`

Le script ne doit être activé que en prod
Pour cela, calculer une const `GTM_ID` en début de fichier:
```ts
const GTM_ID = process.env.ENV_NAME === "prod" ? "GTM-XXXXXXXX" : "";
```

Le reste doit être passé directement dans `defineNuxtConfig` comme suit:
```ts
export default defineNuxtConfig({
    app: {
    head: {
        script: GTM_ID
        ? [
            // ...
        ]
        : [],
        noscript: GTM_ID
        ? [
            // ...
        ]
        : [],
    },
    },
});
```

Voici les informations relative à l'ajout de GTM dans le projet Nuxt :




<!-- PASTER HERE-->