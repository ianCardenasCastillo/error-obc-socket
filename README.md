# error-obc-socket

**myproject/settings.py**
```
WSGI_APPLICATION='myproject.wsgi.application'
ASGI_APPLICATION='myproject.asgi.application'
ASGI_THREADS=1000
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            "hosts": [(os.environ.get('URL_REDIS', 'redis'), os.environ.get('PORT_REDIS', 6379))],
            'capacity': 300
        },
    },
}
```
**myproject/asgi.py**
```
import os

from .wsgi import *

from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter,URLRouter
from django.core.asgi import get_asgi_application
import myapp.routing

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'orbiconnect.settings')

# application = get_asgi_application()
application = ProtocolTypeRouter({
    "http": get_asgi_application(),
    "websocket": AuthMiddlewareStack(
        URLRouter(
            myapp.routing.websocket_urlpatterns
        )
    ),
})
```
**myapp/routing.py**
```
from django.urls import re_path


from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/noticia/', consumers.NoticiaConsumer.as_asgi()),
]
```
**myapp/consumers.py**
```
class NoticiaConsumer(WebsocketConsumer):
    # Funcion cuando se conectan al socket
    def connect(self):
        self.group_name = 'noticias' # Defino el nombre del grupo en este caso noticias
        # 
        async_to_sync(self.channel_layer.group_add)(self.group_name, self.channel_name) # Agregro al usuario que se conecto a este grupo
        self.accept() # Aceptamos su conexion
    def receive(self, text_data):
        pass
    # función de desconexion del socket
    def disconnect(self, close_code):
        # self.close() # Cerramos la conexion del socket
        # async_to_sync(self.channel_layer.group_discard)("noticias", self.channel_name)
        async_to_sync(self.channel_layer.group_discard)(
        'noticias',
        self.channel_name
        )
        raise StopConsumer()
```

**requirements.txt**
* Django>=3.1
* psycopg2-binary>=2.8
* djangorestframework
* pandas
* openpyxl
* xlwt
* xlrd
* django-cors-headers
* requests
* xlsxwriter
* channels
* channels_redis
* Twisted[tls,http2]
