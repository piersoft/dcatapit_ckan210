# dcatapit_ckan210

versione di https://github.com/geosolutions-it/ckanext-dcatapit/ con le patch per CKAN 2.10


appunti sparsi per installazione completa:

Installazione su Raspberry CKAN2.10

https://docs.ckan.org/en/latest/maintaining/installing/install-from-source.html

Non dimenticarsi :
sudo mkdir -p /var/lib/ckan/default
sudo chown www-data /var/lib/ckan/default
sudo chmod u+rwx /var/lib/ckan/default

Idem per —> 
/var/lib/ckan/default/storage/uploads/admin

ckan.extra_resource_fields = themes subthemes license_type resource_license organization_region_it dcat_theme

In ordine:
Installare

HARVEST
DCAT
MULTILANG (non mettere multilang_resource né multilang_harvester nel plugin come indicato erroneamente)
ckan --config=/etc/ckan/default/ckan.ini multilang initdb 

Bug segnalato per permesso negato ai file di traduzione —> ckan -c /etc/ckan/default/ckan.ini translation js
Dopo gli inserimenti nel manageschema (nano /var/solr/data/ckan/conf/managed-schema) di solr riavviare con 
sudo service solr restart
sudo service supervisor restart

DCATAPIT in DEV MODE

pip install routes
Modificare requirements.txt —>  markupsafe==2.0

git clone https://github.com/geosolutions-it/ckanext-dcatapit.git
cd ckanext-dcatapit
pip install -e .
pip install -r dev-requirements.txt

Pip install pylons

Aggiungere:
import model
from pylons import request, config, tmpl_context as c
 in /usr/lib/ckan/default/src/ckan/ckan/lib/base.py

Correggere in /usr/lib/ckan/default/lib/python3.10/site-packages/pylons/controllers/core.py
Tutti gli exept con la ( al posto della virgola.
Esempio —> except HTTPException, httpe: diventa except HTTPException(httpe):

L’errore di mancanza modulo xmlrpc

import xmlrpclib
Replace that with this:
import xmlrpc.client
In /usr/lib/ckan/default/lib/python3.10/site-packages/pylons/controllers/xmlrpc.py

XMLRPC_MAPPING = ((basestring, 'string'), (list, 'array'), (bool, 'boolean'),
NameError: name 'basestring' is not defined
Cambiare in str e cambiare il client cosi:
XMLRPC_MAPPING = ((str, 'string'), (list, 'array'), (bool, 'boolean'),
                  (int, 'int'), (float, 'double'), (dict, 'struct'),
                  (xmlrpc.client.DateTime, 'dateTime.iso8601'),
                  (xmlrpc.client.Binary, 'base64'))

Correggere in /usr/lib/ckan/default/lib/python3.10/site-packages/pylons/controllers/jsonrpc.py
except JSONRPCError, e: —> except JSONRPCError(e):

Solo dopo fare ckan --config=/etc/ckan/default/ckan.ini dcatapit initdb

Commentare in /usr/lib/ckan/default/src/ckanext-dcatapit/ckanext/dcatapit/commands/vocabulary.py le righe 139 a seguire della funzione split locale offered

Solo adesso si possono Importare gli RDF come vocabolari
ckan -c /etc/ckan/default/ckan.ini dcatapit load --filename=vocabularies/languages-filtered.rdf
Ect

sostituire i files mapping plugin helper vocabulary .py in dcatapit dir

Errore 504 catalog.ttl
/usr/lib/ckan/default/src/ckanext-dcatapit/ckanext/dcatapit/dcat/profiles.py
Riga 1209 aggiungere if isinstance(langs, str):



DEMO con harvesting misto (RDF e tramite API) su https://www.piersoft.it/ckan/index2.php 

