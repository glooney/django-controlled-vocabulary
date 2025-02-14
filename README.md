[![PyPI version](https://badge.fury.io/py/django-controlled-vocabulary.svg)](https://badge.fury.io/py/django-controlled-vocabulary)

# Django Controlled Vocabulary

This app provides models and admin interface to link your data to standard vocabularies (e.g. ISO language codes, Wikidata). Benefits: increases the consistency and understandability of your project data.

Development Status: **Alpha** (mostly functional, work in progress)

<img src="docs/img/controlled-term-widget.png" />

_A ControlledTerm field in the Django admin interface. The user selects the vocabulary (here: Wikidata), then starts typing a term in the text box. Suggestions are brought from Wikidata. When the user saves the changes, information about the selected term is copied into the database (url, identifier, label)._

# Features

* create your own controlled lists of terms (i.e. **local** lists)
* look up terms from **remote** vocabularies (i.e. authority files)
* extensible **plug-in architecture** for lookups into particular vocabularies:
  * built-in vocabulary plug-ins, such as ISO 639-2 (Language codes), DCMI Type (Dublin Core resource types)
  * see below for full list
* **stores** used terms from remote vocabularies into your database:
  * space efficient (doesn't clutter the database with unused terms)
  * self-contained (i.e. can still works offline & DB always 'semantically' complete)
* **autocomplete** widget for Django admin; reusable ControlledTermField for your models
* **command line tool** to download vocabulary files from authoritative sources
* [TODO] possibility to store additional **metadata** (e.g. geographic coordinates)
* [TODO] simple **rest API** to publish your own terms

# Data Model & Software Design

## Django models

| Vocabularies | Terms |
| ------------- | ------------- |
| <img src="docs/img/controlled-vocabulary-list.png" width="400" />  | <img src="docs/img/controlled-term-list.png" width="400" />  |

* **ControlledVocabulary**
  * prefix: the vocabulary standard prefix, see http://prefix.cc/wikidata
  * label: the short name of the vocabulary
  * base_url: the url used as a base for all terms in the vocabulary
  * concept: the type of terms this vocabulary contains
  * description: a longer description

* **ControlledTerm**
  * termid: a unique code for the term within a vocabulary
  * label: standard name for the term
  * vocabulary: the vocabulary this term belongs to

## Vocabulary plug-ins / managers

A Vocabulary **plug-in** / **manager** is a python class that provides services for a vocabulary:
* autocomplete terms from local or remote datasets (see ControlledTermField)
* supplies metadata for the vocabulary (see ControlledVocabulary)

Managers can provide terms from a CSV file downloaded from an authoritative source.

Some vocabularies can contain thousands of terms or more. A plugin will
only insert the terms used by your application. The rest will be accessed on
demand from a file on disk or in a third-party server. This approach saves
database space and keeps your application data self-contained.

This project comes with built-in plugins for the following vocabularies:

**ISO 639-2, DCMI Type, Wikidata, FAST Topics, MIME, Schema.org, VIAF, FAST Forms and Genres**

Those plugins are **enabled** by default; see below how to selectively enable them.

This architecture allows third-party plugins to be supplied via separate
python packages.

# Limitations
* **controlled list** rather than fully fledged vocabularies, (i.e. just a bag of terms with unique IDs/URIs, no support for taxonomic relationships among terms like broader, narrower, synonyms, ...)
* no notion of granularity (e.g. geonames country, region, city, street are all treated as part of the same vocabulary)

# Setup

## Installation

Install into your environment:

```
pip install django-controlled-vocabulary
```

Add the app to the INSTALLED_APPS list in your Django settings file:

```
INSTALLED_APPS = [
    ...
    'controlled_vocabulary',
    ...
]
```

Add the following path to your project urls.py:

```
from django.urls import path, include
...

urlpatterns = [
    ...
    path('vocabularies/', include('controlled_vocabulary.urls')),
    ...
]
```

Run the migrations:

```
./manage.py migrate
```

Download vocabulary data and add metadata to the database:

```
./manage.py vocab init
```

## Configuration

### Enabling vocabulary plug-ins

Add the following code in your settings.py to enable specific vocabularies based on the import path of their classes.

```
# List of import paths to vocabularies lookup classes
CONTROLLED_VOCABULARY_VOCABULARIES = [
    'controlled_vocabulary.vocabularies.iso639_2',
    'controlled_vocabulary.vocabularies.dcmitype',
]
```

After enabling a new plug-in / manager, always run `./manage.py vocab init`.

### ControlledTermField

To define a field with an autocomplete to controlled terms in your Django Model, use the following field:

```
from controlled_vocabulary.models import ControlledTermField

...

class MyModel(models.Model):

    ...
    language_code = ControlledTermField(
        'iso639-2',
        null=True, blank=True
    )
```

Where 'iso639-2' is the prefix of a controlled vocabulary in your database.

# vocab (command line tool)

vocab is a django command line tool that lets you manipulate the vocabularies
and the plugins. To find out more use the help:

```
./manage vocab help
```

