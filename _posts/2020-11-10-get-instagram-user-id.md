---
layout: post
category: web scraping
title: Obtener Instagram ID de un usuario sin Instagram API
---

Al momento de realizar una aplicación que utilice datos de los usuarios de Instagram, nace la necesidad de trabajar constantemente con los nombres de usuarios y el ID asociado a estos. Para obtener fácilmente el ID de un usuario a partir de su username o viceversa, el camino más sencillo es utilizar la API de Instagram. Sin embargo, hoy en día la API presenta muchas limitaciones, por lo que muchas veces obtener los datos por nuestra cuenta pasa a darnos más potencial.  

En primer lugar, ¿por qué existe el Instagram ID?, ¿qué es?, el Instagram ID es un número entero único que identifica a cada usuario. No puede haber dos usuarios con el mismo ID. El username parece tener estas mismas características, sin embargo un usuario puede cambiar de username, pero nunca podrá cambiar de ID.


### Pasos previos

Antes de comenzar, se recomienda que utilice alguna extensión para visualizar en nuestro navegador las salidas JSON de manera más legible y con algunas herramientas que facilitarán la interpretación. En mi caso utilizo [JSON Viewer](https://chrome.google.com/webstore/detail/json-viewer/gbmdgpbipfallnflgajpaliibnhdgobh?hl=es) en el navegador Google Chrome, se puede visualizar la salida en la imagen anterior.


### ¡Comencemos!

Recientemente, Instagram se ha actualizado y ahora para obtener el ID requerimos tener una cuenta de Instagram y haber iniciado sesión, lo que complica un poco las cosas. Lo que haremos será iniciar sesión y luego dirigirnos a un link el cual enviará una petición GET a los servidores de Instagram y podremos visualizar la respuesta en nuestro navegador. Esto mismo lo podemos hacer sin iniciar sesión, pero funcionará muy pocas veces y luego Instagram nos bloqueará la petición.  
> El link tiene la siguiente forma:  
`https://www.instagram.com/username/?__a=1`

Donde username corresponde al nombre de usuario del cual queremos obtener su ID asociado.  

Una vez ingresemos al link, podremos ver la respuesta recibida, la cual será un objeto JSON similar al siguiente.

```
{
  ...
  "graphql": {
    "user": {
      "biography": "#YoursToMake",
      "blocked_by_viewer": false,
      "restricted_by_viewer": false,
      "country_block": false,
      "external_url": "http://help.instagram.com/",
      "external_url_linkshimmed": "https://l.instagram.com/?u=http%3A%2F%2Fhelp.instagram.com%2F&e=ATMcekGmICWJC-dpUuLwIeT2f1uJu7h0pSlQ0SBXGn4XVWsW2XG3nLs97oNNQTwT5F5nnPBHPD8z5mYk-mJUi04&s=1",
      "edge_followed_by": {
        "count": 455737510
      },
      "fbid": "17841400039600391",
      "followed_by_viewer": true,
      "edge_follow": {
        "count": 94
      },
      "follows_viewer": false,
      "full_name": "Instagram",
      "has_ar_effects": false,
      "has_clips": true,
      "has_guides": true,
      "has_channel": false,
      "has_blocked_viewer": false,
      "highlight_reel_count": 16,
      "has_requested_viewer": false,
      "hide_like_and_view_counts": false,
      "id": "25025320",
       ...
}
```
Analizándolo un poco podemos observar que el ID del usuario se encuentra en el atributo **graphql.user.id**. Además de esto podremos visualizar mucha información extra, la cual puede ser de gran ayuda. En publicaciones siguientes, hablaremos más sobre estos datos.


### Obteniendo la cookie sessionid

Para realizar la petición desde Python deberemos tener una sesión iniciada. Para ello tenemos dos opciones:
1. Iniciar sesión a una cuenta de Instagram desde Python y utilizar las cookies que obtengamos para las peticiones posteriores.
2. Iniciar sesión a una cuenta de Instagram desde nuestro navegador, copiar las cookies necesarias de la sesión y enviarlas en cada petición realizada.  

En nuestro caso, optaremos por la opción (2).  

Para obtener la cookie sessionid desde nuestro navegador, en primer lugar debemos iniciar sesión. Luego, nos dirigimos a la página web de Instagram y abrimos las “herramientas para desarrolladores”. Tanto en Google Chrome como en Firefox podemos realizar esto presionando F12. Una vez abierta las herramientas, en Google Chrome, podremos ver las cookies en application -> storage -> cookies. Allí solo consideramos las cookies de Instagram e ignoramos cookies de sitios web terceros, si es que existen. En Firefox, podemos ver las cookies en storage -> cookies.


### Automatizando el proceso

Si bien obtener el ID puede que no sea demasiado útil por si solo para crear una aplicación, la demostración de cómo realizarlo puede que sirva como punta pie inicial para comenzar tu proyecto. Además una vez se obtiene el ID, el proceso para obtener la foto de perfil u otros datos es muy similar.  

Para realizar esta tarea, optamos por Python. Python es una alternativa muy potente por su sencillez y facilidad, con pocas líneas podemos realizar mucho. En nuestro caso utilizaremos Python 3.x junto a la librearía requests, para obtener aún mayor sencillez en nuestro código.

```
import requests

username = input("Target username: ")

cookies = {
   "sessionid": sessionid
}

res = requests.get("https://www.instagram.com/" + username + "/?__a=1", cookies = cookies)

if req.status_code == 200:
   id = req.json()["graphql"]["user"]["id"]
   print("El ID de %s es %s" % (username, id));
else:
   print("Error %d al realizar la petición." % (req.status_code))
```

Sencillo, ¿no?  

Realizando una pequeña modificación podemos obtener el enlace a la imagen de su perfil de Instagram. El nombre de este atributo es **profile_pic_url**, aunque en **profile_pic_url_hd** tenemos la imagen con mayor calidad. Una vez obtenido el enlace, descargaremos la imagen.

```
import requests

username = input("Target username: ")

sessionid = input("Session id: ")
​
cookies = {
   "sessionid": sessionid
}

res = requests.get("https://www.instagram.com/" + username + "/?__a=1", cookies = cookies)

if res.status_code == 200:
   datajson = res.json()
   id = datajson["graphql"]["user"]["id"]
   profile_url = datajson["graphql"]["user"]["profile_pic_url_hd"]
   res_img = requests.get(profile_url)
   img = open("profile_" + username + ".jpg", "wb")
   img.write(res_img.content)
   img.close()

   print("Se ha descargado la imagen de perfil de %s con ID %s" % (username, id))
else:
   print("Error %d al realizar la petición." % (res.status_code))
```

La imagen se guardará en el directorio actual con el nombre de *profile_* concatenado al *username* ingresado.


### ¿Qué sigue?

Y esto es todo por ahora, en próximas publicaciones se mostrará como descargar todo el contenido multimedia asociado a un usuario (un poco más complejo que lo realizado en esta entrega). Más adelante, también se explicará cómo obtener una lista de los seguidores de un usuario y otra lista de los usuarios que un usuario sigue.