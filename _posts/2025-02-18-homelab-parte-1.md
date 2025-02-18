---
title: "Parte 1: Construyendo un servidor casero"
date: "2025-02-18"
categories: [homelab]
---

# Creando un servidor casero (HomeLab): Parte 1

¡Bienvenido a una serie de artículos donde aprenderás a montar tu propio servidor, también conocido como Homelab! Esta guía está diseñada tanto para entusiastas de la tecnología como para quienes están dando sus primeros pasos en este mundo, o simplemente para aquellos frikis como yo que disfrutan experimentando.

En esta primera entrada, compartiré contigo el proceso que seguí para construir mi homelab, destacando los aciertos y errores que cometí en el camino. Mi objetivo es que puedas aprender de mi experiencia para que tú no tengas que cometer los mismos errores y puedas crear tu propio homelab de forma más eficiente. Gasolina, y arrancamos!

## Lo primero de todo. Por que construir mi propio homelab?

¡Buena pregunta! La respuesta corta sería: ¿y por qué no? (Vale, es broma).  

Existen muchas razones para montar un homelab. Tal vez tengas un NAS Synology que ya no da abasto con la carga de tus contenedores Docker. O quizás te preocupa la privacidad y prefieres que tus datos no estén en manos de terceros. O puede que, como en mi caso, sea una combinación de ambas cosas, sumado al cansancio de pagar suscripciones por absolutamente todo: Netflix, música, almacenamiento...  

Seamos sinceros: con el paso de los años, todo se ha trasladado al *streaming*. Y cada vez poseemos menos. Tus archivos, tus series, tu música… ya no te pertenecen, están en la *nube*, bajo el control de empresas que pueden cambiar precios, condiciones o incluso eliminar contenido cuando les parezca.

Construir mi propio homelab signficaba recuperar el control. Era tener la libertad de gestionar mis propios datos, alojar mis propios servicios y dejar de depender de terceros.

Bien, esa fue una de las razones por las que decidí construir un homelab. Pero no te voy a engañar: siempre he sido un cacharrero de la tecnología, así que realmente solo necesitaba una excusa para hacerlo. Lo que no sabía es que esa “excusa” acabaría convirtiéndose en un motivo firme y de peso conforme más vueltas le daba.

## Que usos tendría mi homelab?

Una vez resuelta la pregunta del por qué, tocaba responder al qué: ¿qué iba a almacenar y para qué iba a usarlo?

Si has estado atento al párrafo anterior, ya habrás intuido algunas de mis motivaciones, pero aquí te las resumo de manera más concreta:

**Mi propio Netflix** 🎬:

Siempre quise tener una biblioteca personal de películas y series sin depender de suscripciones. Pero pronto descubrí que mi homelab no solo serviría para esto, sino también para ver toda la galeria de fotos y vídeos que guardaba en la nube.

**Entorno de desarrollo** 💻:

Me gusta programar y aprender nuevos lenguajes o frameworks, así que quería un entorno que me permitiera desarrollar localmente y aprender de CI/CD. Además, me interesaba tener un backup de mis repositorios de GitHub para no depender exclusivamente de la nube.

**Backups, backups y más backups** 📂:

Nunca hay suficientes copias de seguridad. ¿Sabes esas fotos que guardas en la galería, que nunca ves pero tampoco quieres borrar porque son recuerdos? Exacto, mi homelab tenía que encargarse de almacenar y proteger todo eso.

**Domótica centralizada** 🏠:

Alexa está muy bien… hasta que tienes 20 dispositivos de distintas marcas y necesitas 20 apps para controlarlos. Eso no iba conmigo. Quería unificar todo bajo un solo sistema y gestionar mi casa de forma más eficiente.

**Aprendizaje IT y seguridad informática**:

Si eres administrador de sistemas, trabajas en IT o simplemente te apasiona la tecnología, montar un homelab es una de las mejores inversiones que puedes hacer. ¿Por qué? Porque te permite experimentar con nuevas tecnologías, entender cómo funcionan y aprender a aplicarlas en el mundo real.

Un homelab es un entorno perfecto para probar configuraciones, simular infraestructuras y desarrollar habilidades sin riesgo. Ya sea para mejorar tu seguridad informática, aprender sobre virtualización, redes o automatización, contar con tu propio laboratorio te da un impulso enorme en conocimientos prácticos.

Básicamente, montar mi homelab fue un billete de ida al aprendizaje continuo en el área de IT. 🚀

## Componentes del homelab:

En honor a la verdad, fui algo cutre con el gasto. Tras ocho años de uso, decidí renovar mi viejo PC Gaming, que aún montaba una GTX 1050 Ti. Teniendo en cuenta que ya vamos por la serie RTX 5000, era evidente que ya tocaba hacer algún cambio.

Así que sí, la idea del homelab surgió después de actualizar mi PC Gaming y no saber qué hacer con las piezas sobrantes. No quería venderlas, porque sabía que todavía podían tener un buen uso. La realidad es que ya no tengo tanto tiempo para jugar como antes, pero de vez en cuando me gusta desconectar un rato.

Aprovechando el hardware viejo, logré abaratar bastante los costes de mi servidor casero:

- GTX 1050 Ti
- AMD Ryzen 5 2600X
- 16GB RAM DDR4 2400MHz
- 4TB HDD (x1 o varios discos, según el caso)

No es la configuración más potente ni eficiente que existe, pero para un servidor ligero que no requería grandes recursos de cálculo, era más que suficiente. Solo tuve que comprar tres componentes nuevos: una caja, una placa base y una fuente de alimentación de 500W. Tenía claro que quería un servidor pequeño y funcional, así que opté por el formato mATX, que me ofrecía el equilibrio ideal entre tamaño, rendimiento y consumo.

**¿Es la opción más eficiente?** No. Ni la gráfica ni el procesador están diseñados para un entorno de servidores, y lo sé. Pero, siendo sinceros, lo importante aquí era la cartera y el presupuesto. Quería algo que fuera barato, funcional y que ofreciera la mejor relación calidad/precio dentro de sus posibilidades.

## El sistema operativo  

Vale, aquí es donde más de uno se lleva las manos a la cabeza… ¿Qué sistema operativo elegí? Pues, por sorprendente que parezca… **Windows**.  

### ¿Por qué Windows y no Ubuntu Server u otra distribución Linux?  

Buena pregunta. Inicialmente, mi idea era instalar **Ubuntu Server** sin interfaz gráfica, ya que trabaja de maravilla incluso en PCs con pocos recursos. Sin embargo, había un problema, y uno bastante gordo, que me *obligó* a decantarme por Windows:  

**iCloud**  

Y, ¿qué tiene que ver iCloud en todo esto? Pues veréis, como diría Jack el Destripador… vayamos por partes.  

Uno de mis principales objetivos era hacer copias de seguridad **locales** de **iCloud** en mi homelab. Pero, como era de esperar, Apple no lo pone nada fácil (al menos en Linux). Tras investigar un poco, encontré un proyecto en **GitHub** que permitía realizar backups de iCloud a través de un contenedor Docker.  

Pero aquí surgían tres problemas importantes:  

- **El proyecto no era oficial** → Dependía del desarrollador o la comunidad. Si dejaban de mantenerlo, en algún momento dejaría de funcionar.  
- **Requería delegar acceso total a mi cuenta de iCloud** → Necesitaba generar una API Token con acceso completo a mi cuenta y confiar en que el código era seguro. Aunque hoy lo fuera, nada garantizaba que en el futuro no se volviera malicioso.  
- **No era compatible con ADP (Advanced Data Protection)** → Si quería usarlo, debía desactivar una de las funciones de seguridad más importantes de iCloud.  

Si alguien tiene curiosidad, este es el proyecto: https://github.com/boredazfcuk/docker-icloudpd.git

### **Vale, pero… ¿qué es ADP?**  

iCloud ofrece una opción llamada **ADP (Advanced Data Protection)**, que permite activar el cifrado **de extremo a extremo** en toda la información almacenada en la nube. Con esta opción habilitada, ni siquiera Apple puede acceder a mis datos, lo que supone una capa extra de seguridad ante posibles filtraciones.  

Desactivar ADP solo para poder hacer backups locales de **mis propios datos** no era una opción.  

Así que, antes de comprometer la seguridad de mi cuenta, decidí probar otra solución: **Windows**.  

## **¿Y qué tiene que ver Windows en todo esto?**  

Aquí está la clave: **el cifrado extremo a extremo** de ADP hace que solo los **dispositivos autenticados** puedan acceder a los datos. Y, entre esos dispositivos, Apple incluye… **PCs con Windows**.  

Sí, aunque parezca irónico, Apple desarrolló un programa llamado **iCloud for Windows**, que permite sincronizar iCloud Photos, iCloud Drive y otros servicios en un PC con Windows autenticado.  

Esto no significa que descargue automáticamente los archivos al disco local, sino que simplemente permite **acceder** a ellos en la nube. Solo cuando abres un archivo (por ejemplo, una foto o un documento), Windows lo descarga en ese momento.  

### **¿Cómo hago los backups?**  

Eso lo explicaré en detalle en otro post, así que si quieres saber cómo conseguí hacer copias locales de iCloud sin desactivar ADP… tendrás que estar atento. Pero ya te he dejado una pista ;)

Aunque para no dejarte con un mal sabor de boca, aquí va otro motivo por el que elegí Windows:  

## **WSL2: La solución definitiva**  

Además de resolver mi problema con iCloud, recordé que Windows cuenta con **WSL2 (Windows Subsystem for Linux - v2)**.  

WSL2 permite ejecutar un entorno Linux dentro de Windows sin necesidad de una máquina virtual independiente ni arranque dual.  

Como dice la documentación oficial de Microsoft:  

> *WSL está diseñado para proporcionar una experiencia perfecta y productiva para los desarrolladores que quieren usar Windows y Linux al mismo tiempo.*  

Exactamente lo que necesitaba.  

Gracias a WSL2, pude seguir usando **Windows** para gestionar las copias de seguridad de **iCloud** y, al mismo tiempo, mantener mis servicios en **Linux** dentro de un entorno WSL2. No era la solución que había planeado inicialmente, pero… **funcionaba**. Y eso era lo que importaba.

## El gran olvidado: Proxmox

Proxmox. La solución de virtualización por excelencia para homelabs domésticos (y no tan domésticos). ¿Por qué no lo usé?

La verdad, podría haberlo instalado. Seguramente me habría facilitado la vida en algunos aspectos y habría mejorado mucho la seguridad. Y no descarto hacerlo en un futuro.

El problema principal era la memoria RAM. Para cada máquina virtual en Proxmox, hay que asignar una cantidad fija de memoria. Y recordemos que solo tengo 16GB.

Aunque de todas formas necesitaba Windows para iCloud, con Proxmox habría podido crear una máquina virtual para cada cosa:

Windows → Para gestionar iCloud.
Linux → Para el resto de servicios.
Pero aquí venía el problema: cada VM necesitaría, al menos, 8GB de RAM para funcionar bien. Si sumamos que mi máximo era 16GB, el margen de maniobra era prácticamente nulo.

A pesar de esto, la configuración que hice es funcional y cumple su propósito. No descarto instalar Proxmox en el futuro, especialmente cuando consiga más memoria RAM y más almacenamiento. En ese momento, podría aprovechar mejor VLANs, mejorar la seguridad, hacer backups de VMs, y un sinfín de cosas más.

## Conclusiones

Si hay algo que quiero que saques de todo esto, es lo siguiente: no esperes al momento "perfecto" o al equipo "perfecto" para montar tu homelab.

Con plataformas como Proxmox y una comunidad enorme que respalda un sinfín de servicios en contenedores, puedes empezar poco a poco e ir escalando a tu propio ritmo. Un homelab no es solo un conjunto de servidores: es un espacio para aprender, experimentar y mejorar habilidades técnicas.

Es un proyecto que crece contigo. Y lo mejor de todo es que nunca es tarde para empezar.

Da igual si usas hardware de segunda mano, un viejo PC o incluso una Raspberry Pi. Lo importante es dar el primer paso. Créeme, una vez te metas en este mundo, te preguntarás por qué no lo hiciste antes.

## Contacto

Recuerda que si quieres contactar conmigo puedes hacerlo a través de mi correo [ablanco@cyberit.es](mailto:ablanco@cyberit.es) o visitando mi página web [CyberIT](https://cyberit.es)