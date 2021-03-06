Available in [Spanish](https://github.com/AppCivico/pegabot-backend/tree/master/README_ES.md) [Portuguese](https://github.com/AppCivico/pegabot-backend/tree/master/README_PT.md)

# spottingbot
Analyzing profiles on Twitter to detect bot behavior

**_spottingbot is an experimental and open-source project that needs you to evolve, do not hesitate to contribute on our [GitHub repository](https://github.com/AppCivico/pegabot-backend) by opening a pull request or to contact us at [lucas.ansei@appcivico.com](mailto:lucas.ansei@appcivico.com). Documentation on how the current indexes are calculated is also available [here](https://github.com/AppCivico/pegabot-backend/tree/master/documentation)_**

**You can also join us on our [Telegram group](https://t.me/joinchat/AOHjCkUyx1zPuNzhf36mEw) to freely talk about suggestions, improvement or simply ask us anything**

## Usage

spottingbot can be used both as a command-line interface application (cli) or as an independent module

### Command-line interface

#### Installation

`npm install`

#### Settings file

Create a .env file that contains:

```
TWITTER_CONSUMER_KEY="Your application consumer key"
TWITTER_CONSUMER_SECRET="Your application consumer secret"
TWITTER_ACCESS_TOKEN_KEY="Your application access token key, only for user authentication"
TWITTER_ACCESS_TOKEN_SECRET="Your application access token secret, only for user authentication"

PORT="The port that the app will attach itself"

USE_CACHE="Flag that will control the cache usage. Must be either 0 or 1.
DEFAULT_CACHE_INTERVAL='The interval that the cache will be considered valid. Must be a valid Postgresql format, e.g: '30_days'."

DATABASE_HOST="Your Postgresql host"
DATABASE_USER="Your database user"
DATABASE_PASSWORD="Your database password"
DATABASE_NAME="Your database password"

BULK_SECURITY_TOKEN='Key that will be used to authorize calls to the bulk endpoint'

PUPPETER_SERVICE_ROOT_URL="Url to the service that will take screenshots from the Twitter profile"
PUPPETER_SECRET_TOKEN="Token for the screenshot service"

```
##### PSQL database

Pegabot uses a cache system to avoid remaking an analysis on the same user.
This cache system uses a PostgreSQL database.

How to set-up the PostgreSQL database:

1. Install and set PSQL on your machine
2. Create a database for pegabots. If you want, you can create a new user with a password to own this database, or just use the default postgres user.
3. Fill up the database data on the .env file. Example:

```
DATABASE_HOST="127.0.0.1"
DATABASE_USER="postgres"
DATABASE_PASSWORD=""
DATABASE_NAME="pegabot"
```

4. Install the npm module `sequelize-cli`
5. Run migrations with `sequelize-cli db:migrate`

## Starting the API
You can run the command: `npm run dev` in order to start with nodemon and Babel. Or you can start with `npm run start`, which will build the application and run from a `/build` folder without Babel.

## Endpoints

### `/botometer`
Get analysis with only main indexes.

**Method**: `GET`

**Example**: `curl --request GET --url 'http://127.0.0.1:5010/botometer?profile=twitter&search_for=profile&is_admin=true'`


**Required parameters**
- `profile` (STRING): user handle that will be analyzed;

- `search_for` (STRING): only one value allowed `profile`

**Optional parameters**
- `is_admin` (BOOLEAN): used to identify requests that are coming from the Pegabot admin interface; (1|0 OR true|false)

**Response**
```
{
  "metadata": {
    "count": 1
  },
  "profiles": [
    {
      "username": "twitter",
      "url": "https://twitter.com/twitter",
      "avatar": "http://pbs.twimg.com/profile_images/1354479643882004483/Btnfm47p_normal.jpg",
      "language_dependent": {
        "sentiment": {
          "value": 0.57
        }
      },
      "language_independent": {
        "friend": null,
        "temporal": 0.32269754443513865,
        "network": 0.4103015075376884,
        "user": 0
      },
      "bot_probability": {
        "all": 0.18614272171040389,
        "info": "<p>Um dos crit??rios que mais teve peso na an??lise foi o ??ndice de Perfil</p>"
      },
      "user_profile_language": null
    }
  ]
}
```

### `/analyze`
Get analysis with main indexes and subindexes.

**Method**: `GET`

**Example**: `curl --request GET --url 'http://127.0.0.1:5010/analyze?profile=twitter&search_for=profile'`


**Required parameters**
- `profile` (STRING): user handle that will be analyzed;

- `search_for` (STRING): only one value allowed `profile`

**Response**
```
{
  "root": {
    "profile": {
      "handle": "twitter",
      "link": "https://twitter.com/twitter",
      "label": "Perfil",
      "description": "<p>Algumas das informa????es p??blicas dos perfis consideradas na an??lise do PEGABOT s??o o nome do perfil do usu??rio, e quantos caracteres ele possui, quantidade de perfis seguidos (following) e seguidores (followers), texto da descri????o do perfil, n??mero de postagens (tweets) e favoritos. Ap??s coletar as informa????es, os algoritmos do PEGABOT processam e transformam os dados recebidos em vari??veis que comp??em o c??lculo final de probabilidade.</p>",
      "figure": "",
      "analyses": [
        {
          "title": "SELO DE VERIFICA????O",
          "description": "A presen??a do selo de verifica????o oferecido pelo Twitter influencia positivamente nos resultados, uma vez que a plataforma possui um procedimento manual para validar a identidade desses usu??rios.",
          "summary": "<p>Usu??rio verificado</p>",
          "conclusion": 0
        }
      ]
    },
    "network": {
      "description": "<p>O algoritmo do PegaBot coleta uma amostra da linha do tempo do usu??rio, identificando hashtags utilizadas e men????es ao perfil para realizar suas an??lises. O objetivo ?? identificar caracter??sticas de distribui????o de informa????o na rede da conta analisada.</p>O ??ndice de rede avalia se o perfil possui uma frequ??ncia alta de repeti????es de men????es e hashtags. No caso de um bot de spams, geralmente se usam as mesmas hashtags/men????es, e ?? isso que esse ??ndice observa. Por exemplo, se 50 hashtags s??o usadas e s??o 50 hashtags diferentes, n??o ?? suspeito, mas se s?? uma hashtag ?? usada 100% das vezes, ent??o ?? muito suspeito.</p>",
      "label": "Rede",
      "analyses": [
        {
          "title": "DISTRIBUI????O DAS HASHTAGS",
          "description": "<p>Calcula o tamanho da distribui????o dessas hashtags na rede. Ou seja, avalia se a utiliza????o de hashtags do perfil apresenta uma frequ??ncia anormal.</p><p>Quanto mais pr??ximo de 0% menor a probabilidade de ser um comportamento de bot.</p>",
          "summary": "<p>Total: 10. ??nicas: 3</p>",
          "conclusion": "0.70",
          "hashtags": []
        },
        {
          "title": "DISTRIBUI????O DAS MEN????ES",
          "description": "<p>Calcula o tamanho da distribui????o de men????es ao perfil do usu??rio na rede. Ou seja, avalia as men????es realizadas pelo perfil com base em sua frequ??ncia.</p><p>Quanto mais pr??ximo de 0% menor a probabilidade de ser um comportamento de bot.</p>",
          "summary": "<p>Total: 14. ??nicas: 14</p>",
          "conclusion": "0.00",
          "mentions": []
        },
        {
          "title": "HASHTAGS E MEN????ES",
          "description": "HASHTAGS E MEN????ES",
          "summary": "<p>Calculamos o score distr??buido (0.35) e o tamanho da rede (0.06)</p>",
          "conclusion": "0.41",
          "stats": []
        }
      ]
    },
    "emotions": {
      "description": "<p>Ap??s coletar os dados, os algoritmos do PEGABOT fornecem uma pontua????o, em uma escada de -5 a 5m de cada uma das palavras dos tweets coletados. A classifica????o se baseia em uma biblioteca, onde, cada uma das palavras possui uma pontua????o, sendo considerada mais ou menos negativa, positiva ou neutra. Assim, ao final da classifica????o, calcula-se a pontua????o m??dia para a quantidade de palavras positivas, negativas e neutras utilizadas pelo usu??rio.</p>",
      "label": "Sentimentos",
      "analyses": [
        {
          "summary": "<p>o perfil tem pontua????o de 0.57, classificando-se como positivo.</p>",
          "conclusion": 0.57,
          "samples": {
            "title": "VEJA AQUI O EXEMPLO DE 3 TWEETS DO USU??RIO",
            "list": []
          }
        }
      ]
    }
  }
}
```

### `/botometer-bulk`
Bulk process for first endpoint.
**The params for this request must be sent as JSON**

**Method**: `GET`

**Example**:
```
curl --request GET \
  --url http://127.0.0.1:5010/botometer-bulk \
  --header 'Content-Type: application/json' \
  --header 'X-API-KEY: $ENV{BULK_API_KEY}' \
  --data '{
	"is_admin": true,
	"profiles": [
		"twitter"
	] 
}'
```

**Required headers**
- `apiKey` (string): Must be set on the `.env` file, with the `BULK_SECURITY_TOKEN` name;

**Required parameters**
- `profiles` (STRING ARRAY): Array containing the profiles that will be analyzed.
  - **MAX ARRAY SIZE**: 50

**Optional parameters**

- `is_admin` (BOOLEAN): used to identify requests that are coming from the Pegabot admin interface; (1|0 OR true|false)
- `twitter_api_consumer_key` (STRING): Consumer key that will be used to instantiate a Twitter API client;
  - **Must be sent alongside `twitter_api_consumer_secret`**
- `twitter_api_consumer_secret` (STRING): Consumer key that will be used to instantiate a Twitter API client;
  - **Must be sent alongside `twitter_api_consumer_key`**
  
**Response**
```
{
  "analyses_count": 1,
  "analyses": [
    {
      "twitter_user_data": {
        "id": "111111111111111111111111111",
        "handle": "@twitter",
        "user_name": "Twitter",
        "url": "https://twitter.com/twitter",
        "avatar": "https://twitter.com",
        "created_at": "Thu Apr 09 21:32:54 +0000 2020"
      },
      "twitter_user_meta_data": {
        "tweet_count": 2176,
        "follower_count": 304,
        "following_count": 537,
        "hashtags": [],
        "mentions": []
      },
      "pegabot_analysis": {
        "user_index": 0.2755072802592182,
        "temporal_index": 0.09934108527131784,
        "network_index": 0.7706060606060606,
        "sentiment_index": 0.6,
        "bot_probability": 0.34909088522731935
      },
      "metadata": {
          "used_cache": true
      }
    }
  ]
}
```


## Using as a module
*Both User and App-only authentication are supported, for App-only, the Bearer token will be automatically requested*
#### Start

`npm start username`

*or*

`source/cli.js username`

*`username` has to be replaced by the profile to analyze*


#### Install bin locally on your system

`npm link` *sudo might be necessary*

*Then*

`spottingbot username`

### Module

#### Call

```js
const spottingbot = require('spottingbot');

spottingbot(username, index);
```

`username` is a string that contains the screen name of the Twitter profile to analyze.

`twitter_config` is an object that contains Twitter credentials, both User and App-only authentication are supported, for App-only, the Bearer token will be automatically requested, the `twitter_config` object should be like this:

```js
{
  consumer_key: "Your application consumer key",
  consumer_secret: "Your application consumer secret",
  access_token_key: "Your application access token key", // Only for User authentication
  access_token_secret: "Your application access token secret" // Only for User authentication
}
```

`index` is used to disable some indexes, it's an object that looks like this:
```js
{
  user: true,
  friend: true,
  temporal: true,
  network: true
}
```

By default, or if omitted, everything is `true`.

To disable only one index, it isn't necessary to add the others keys in the object, `{friend: false}`, will work.

#### Return value

*spottingbot* handles both *callback* style and *node promise* style

##### Callback

```js
spottingbot(username, twitter_config, index, function(error, result) {
  if (error) {
    // Handle error
    return;
  }
  // Do something with result
})
```

##### Promise

```js
spottingbot(username, twitter_config, index)
  .then(result => {
    // Do something with result
  })
  .catch(error => {
    // Handle error
  })
```

##### Value

The return value is an object that contains

```js
{
  metadata: {
    count: 1 // Always 1 for now
  },
  profiles: [
     {
       username: 'screen_name',
       url: 'https://twitter.com/screen_name',
       avatar: 'image link',
       language_dependent: {
         sentiment: {
           value: 0.65
         }
       },
       language_independent: {
         friend: 0.19,
         temporal: 0.37,
         network: 0.95,
         user: 0
       },
       bot_probability: {
         all: 0.37
       },
       user_profile_language: 'en',
     }
  ]
}
```


**spottingbot is a project inspired by [Botometer](https://botometer.iuni.iu.edu/#!/), an [OSoMe](https://osome.iuni.iu.edu/) project.**

**This project is part of the [PegaBot](http://www.pegabot.com.br) initiative.**

**PegaBot is a project of the [Institute of Technology and Society of Rio de Janeiro (ITS Rio)](https://itsrio.org), [Instituto Equidade & Tecnologia](https://tecnologiaequidade.org.br/) and [AppC??vico](https://appcivico.com/).**

**spottingbot is an experimental and open-source project that needs you to evolve, do not hesitate to contribute on our [GitHub repository](https://github.com/AppCivico/pegabot-backend) by opening a pull request or to contact us at [jordan@appcivico.com](mailto:jordan@appcivico.com). Documentation on how the current indexes are calculated is also available [here](hhttps://github.com/AppCivico/pegabot-backend/tree/master/documentation)_**
