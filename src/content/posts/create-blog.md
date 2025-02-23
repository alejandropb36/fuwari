
---
title: Creación del blog
published: 2025-02-22
description: 'Cómo se creó este blog'
image: ''
tags: [Blogging, Astro, Tech]
category: 'Blogging'
draft: false 
lang: ''
---

# Motivo

Este blog nace de mi pasión por el aprendizaje continuo. Aquí compartiré experiencias, buenas prácticas y soluciones a problemas técnicos que encuentro en mi día a día. Creo en la simplicidad del código, la escalabilidad de las soluciones y el poder del conocimiento compartido.

## Elección de la plataforma

Al momento de crear este blog, consideré varias opciones:

### 1. Desarrollar mi propia plantilla con Astro.js

Astro.js es un framework que me gusta mucho para este tipo de proyectos. De hecho, ya tengo otros proyectos en producción con esta tecnología. Sin embargo, para mi blog no quería invertir demasiado tiempo en la implementación, por lo que buscar una alternativa más rápida cobró mayor sentido.

### 2. Usar herramientas ya existentes como WordPress

Esta opción quedó descartada rápidamente, ya que quería escribir los artículos con un formato lo más minimalista posible, preferiblemente en Markdown (MD). WordPress no se ajustaba del todo a este enfoque.

### 3. Usar una **plantilla preexistente de Astro.js**

Desde el principio, esta opción me pareció la más adecuada. Me permitiría ahorrar tiempo en el desarrollo, aprovechar una tecnología que ya conozco y mantener el contenido en un formato simple como `MD`.

Tras revisar varias plantillas creadas por la comunidad, encontré dos opciones destacadas:

- **[Ovidius Astro Theme - GPL3](https://ovidius-astro-theme.netlify.app/)**
    
    Es una opción minimalista y compatible con la última versión de Astro.js (v5). Sin embargo, su diseño no me convenció del todo. Aun así, es un proyecto interesante en el que me gustaría colaborar en el futuro para agregar nuevas funcionalidades.

- **[Fuwari - MIT](https://fuwari.vercel.app/)**

    Esta fue la opción que elegí. Su diseño me gustó más y fue el factor decisivo para adoptarlo. Aunque no está completamente actualizado a las últimas versiones, puedo contribuir en su desarrollo para mejorar este aspecto.

## Implementación

Gracias a **Fuwari**, pude tener el blog funcionando rápidamente. Actualmente, tengo un fork de este repositorio desplegado con **GitHub Actions** en **GitHub Pages**. Puedes consultarlo aquí:

::github{repo="alejandropb36/fuwari"}

La rama que utilizo es `alejandropb`.

Quiero agradecer a los creadores de este proyecto por su trabajo:

::github{repo="saicaca/fuwari"}
