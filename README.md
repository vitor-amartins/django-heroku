# Como fazer o deploy da sua aplicação Django no Heroku

## 1. Escondendo as configurações da instância

#### Instale o python-decouple

```pip3 install python-decouple```

#### Crie um arquivo chamado *.env* no diretório raíz do seu projeto e coloque as seguintes variáveis:

```SECRET_KEY=COLE_AQUI_A_SECRET_KEY_DO_SEU_PROJETO```

```DEBUG=True```

> Pegue a Secret Key no arquivo *settings.py*. Lembre-se de copiar sem as aspas.

#### Coloque no seu *settings.py* as seguintes linhas:

```python
from decouple import config
```

> As linhas seguintes você deve substituir às variáveis existentes no seu *settings.py*.

```python
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
```

##### O decouple vai ser o responsável por importar essas configurações para o seu projeto.

## 2. Configurando o Banco de Dados

### Instale o dj-database-url

```pip3 install dj-database-url```

#### Coloque no seu *settings.py* as seguintes linhas:

> Substitua a variável  ```DATABASES``` do seu *settings.py* pela seguinte:

```python
from dj_database_url import parse as dburl

default_dburl = 'sqlite:///' + os.path.join(BASE_DIR, 'db.sqlite3')

DATABASES = {
    'default': config('DATABASE_URL', default=default_dburl, cast=dburl),
}
```

##### O dj_database_url vai cuidar da configuração com o banco da sua aplicação no Heroku.

## 3. Configurando arquivos estáticos

#### Instale o dj-static

```pip3 install dj-static```

#### No seu arquivo *wsgi.py*, coloque:

> Substitua a variável  ```application``` do seu *wsgi.py* pela seguinte:

```python
from dj_static import Cling

application = Cling(get_wsgi_application())
```

#### No seu arquivo *settings.py*, adicione ao final:

```python
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

##### O Cling vai gerenciar as requisições à seus arquivos estáticos no Heroku.

## 4. Criando os últimos arquivos necessários

#### Crie o arquivo *requirements-dev.txt* com as depedências do seu projeto:

```pip3 freeze > requirements-dev.txt```

#### Crie o arquivo *requirements.txt* com o seguinte conteúdo:

```bash
-r requirements-dev.txt
gunicorn==19.9.0
psycopg2==2.8.2
```

> As versões do gunicorn e psycopg2 podem ser alteradas. Essas foram as que utilizei no meu último deploy. Consulte as versões mais recentes no [PyPi](https://pypi.org/).

#### Crie o arquivo *Procfile* e adicione o seguinte código:

```bash
web: gunicorn NOME_DO_SEU_PROJETO.wsgi --log-file -
```

> Não esqueça de substituir o nome do seu projeto

#### Crie o arquivo *runtime.txt* e adicione o seguinte código:

```bash
python-3.6.0
```

## 5. Criando o app no Heroku

> Você deve instalar o [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli) antes de continuar

#### Execute o seguinte comando no terminal:

```bash
heroku apps:create app-name
```

> **Salve os dois links que serão gerados no comando anterior**

#### Inclua o primeiro link (provavelmente algo como https://app-name.herokuapp.com/) dentro de ```ALLOWED_HOSTS``` no seu *setings.py* sem o https e sem as barras.

> Exemplo:
> ```ALLOWED_HOSTS = [app-name.herokuapp.com]```

##### Agora iremos exportar as variáveis que salvamos no *.env* para o servidor...

#### Execute os seguintes comandos no terminal:

```bash
heroku plugins:install heroku-config
heroku config:push
```

#### Execute o seguinte comando no terminal e você verá as variáveis que colocou no seu arquivo *.env*:

```bash
heroku config
```

## 6. Publicando o app

#### Execute os seguintes comandos no terminal:

```bash
git add .
git commit -m "Configuring the app"
git push heroku master --force
```

#### E agora execute o seguinte para configurar o banco de dados:

```bash
heroku run python3 manage.py migrate
```

#### Caso deseje criar um superusuário, basta executar:

```bash
heroku run python3 manage.py createsuperuser
```

## Pronto! Seu app já está rodando no Heroku!
