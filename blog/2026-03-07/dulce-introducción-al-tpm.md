---
title: "Dulce introducción al TPM"
authors: jorjai
tags: ["TPM", "Seguridad Informática", "Hardware"]
---

Ha pasado casi un mes desde la última vez que nos vimos por aquí. Os debo decir, sin embargo, que mi ausencia no ha sido injustificada. Los que me conozcáis y los que me sigáis en Linkedin, ya lo sabeis, pero para el resto: *¡Estoy haciendo prácticas en empresa!*

Me ha costado hacerme al ritmo de estudiar y trabajar a la vez, pero creo que ya lo tengo dominado (o eso espero, porque todavía me quedan 3 meses de universidad). Lo bueno de todo esto es que he aprendido un montón de cosas nuevas, la mayoría relacionadas con el TPM. Es posible que a algunos os suene (cierto sistema operativo desarrollado por una empresa que no me cae bien lo pide de manera obligatoria para funcionar), pero para los que no, os voy a contar un poco de qué va esto.

<!--truncate-->

Nos os quiero dar la chapa demasiado porque sé que la mayoría de los que me leéis ni siquiera estáis en este sector, así que voy a hacerlo de la manera que más me gusta, con una analogía.

## El TPM es literalmente tus padres (*más o menos*)

¿Os acordáis de cuando todavía erais txikis (pequeños, en euskera) y no teníais llaves de vuestra casa? Igual erais de esos niños o esas niñas que sí que las tenían, pero yo no. ¿La razón principal? Es evidente, porque a esa edad no éramos lo suficientemente responsables para llevarlas. O dicho de otro modo, desde el punto de vista de los adultos, **no era "seguro" que los niños tuvieran acceso a las llaves de la casa**.

**Un ordenador es como un niño pequeño**, si el atacante es lo suficientemente astuto, puede engañar al ordenador para que le dé acceso a cosas que no debería. Como cuando te montas en esa fourgoneta blanca donde dicen que van a darte caramelos...

Bien, **el TPM es como tus padres**, es el encargado de cuidar de tus llaves y asegurarse de que no caigan en las manos equivocadas. Aunque te metas en la furgoneta y te lleven a un oscuro y húmedo sótano, los secuestradores no podrán entrar en tu casa porque tus padres tienen las llaves y no van a abrir la puerta a nadie que no sea de confianza.

### Entonces, si no tengo llaves, ¿cómo entro a mi casa?

Fácil, tus padres te dejan entrar porque saben que eres tú. De la misma manera, el TPM permite que el ordenador sepa que "eres tú" y te deje acceder a tus datos de forma segura.

De igual manera, si estás en el parque y necesitas subir a tu casa para hacer aguas mayores, les pides a tus padres que te acompañen y que te abran la puerta. Eso sí, luego vuelves a salir y te vas al parque a jugar, que a las maquinitas ya jugarás cuando tus amigos se vayan a sus casas.

## ¿Vale, pero tú qué has estado haciendo?

Gracias por preguntar. Os pongo en contexto. La sede principal de la empresa está en Arrasate, pero como estoy estudiando en Donosti, no puedo ir y volver todos los días. De alguna manera me las apañe para que me dejaran hacer las prácticas en una sede de trabajo remoto que tienen mucho más cerca de mi casa, en Galarreta.

El caso es que solo coincido con mi tutor de prácticas una vez a la semana. Mientras tanto, me va mandando trabajitos. *Léete esto, investiga eso otro, prueba a hacer aquello...*

### Primera semana: leyendo, leyendo y leyendo

El primer día tuve que ir a Arrasate a que me enseñaran todo el edificio y presentarme a la gente. Fliparíais con lo bien que lo tienen montado, una locura (hay hasta una cafetera de las que tienen el los bares).

Mi tutor me explicó que estaba haciendo cosas con el TPM, y me mandó para leerme [un paper](https://www.sciencedirect.com/science/article/pii/S266628172300015X) bastante chulo sobre cómo se podía extraer la clave maestra de BitLocker espiando el bus de comunicación entre el TPM y la CPU.

Durante esa semana estuve leyendo, tomando notas, viendo vídeos sobre cómo interactuar con el TPM, y algunas cosillas más. Fue más que nada empezar a enterarme de qué iba la cosa, porque lo guay llega después. Voy avisando, aquí la cosa se pone un poco más técnica.

:::warning Sección en progreso → falta:

- Explicar operaciones básicas del TPM (PCRs, NV Storage, etc.)
- Explicar cómo se puede interactuar con el TPM (TPM2.0 Tools, etc.)
- Explicar más o menos el ataque 
:::

### Segunda semana: montando un entorno de pruebas

El lunes de la siguiente semana me explicó "el problema" que estaba teniendo en uno de sus proyectos.

