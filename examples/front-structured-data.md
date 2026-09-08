Fait un état des lieux des données structurées du frontend
J'ai tester avec l'outil validator.schema.org et il en manque
Ajouter les partout en front pour les : breadcrumb, articles, FAQ, ...

Ils doivent tous utiliser une base commun de code maintenance
Si il en existe déjà, migre les.

Par exemple, pour le breadcrumb :
```javascript
const breadcrumbJsonLd = computed(() => ({
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  itemListElement: props.items.map((item, index) => ({
    "@type": "ListItem",
    position: index + 1, 
    name: item.name,
    ...(item.url ? { item: item.url } : {}),
  })),
}));

useHead({
    script: [
        {
            type: "application/ld+json",
      innerHTML: JSON.stringify(jsonLd),
    },
  ],
});
```
