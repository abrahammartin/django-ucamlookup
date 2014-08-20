# Introduction

django-ucamlookup is a library which provides useful methods and templates to integrate your 
[Django](https://www.djangoproject.com/) application with the University of Cambridge University 
[Lookup service](https://www.lookup.cam.ac.uk/). 

## Use

Install django-ucamlookup using pip:

```bash
pip install django-ucamlookup
```

Add django-ucamlookup to your installed applications in your project configuration settings.py:

```python
    INSTALLED_APPS=(
    ...
        'ucamlookup', 
    ...
    ),
```

and the urls entries in the urls.py file:

```python
    urlpatterns = patterns(
    ...
        # lookup/ibis urls
        url(r'^ucamlookup/', include('ucamlookup.urls')),
    ...
    )
```

## Requirements and warning

This module will only work inside the University of Cambridge network. Make sure your users are authenticated as 
University of Cambridge users (you can use the django-ucamwebauth module) or you will be exposing personal data to 
non authorised users.

## Lookup User

django-ucamlookup modifies a User object each time is going to be saved, either new or update, and assigns to its 
*last_name* property the visible name from lookup for that user. The username is used to search for this user in lookup.

## Lookup Group

django-ucamlookup includes a new model called LookupGroup that it is used to cache lookup models. It is used to store
the lookup group id and its name, and therefore used to reduce the number of call to the lookup service. It can also be
used to create relation with other models. For example, let's say we have a model called Secret and we only want to let
access to it to users inside a certain group or groups. We will create a ManyToMany relation from Secrets to 
LookupGroup.

The name of the group is retrieve from the lookup service each time the group is saved (new or updated). The name is
stored in the *name* property of the class and the id of the lookup group is stored in *lookup_id*.

## Template macros

Four macros are available to be used in a template: ucamlookup_user, ucamlookup_group, ucamlookup_users, and 
ucamlookup_groups. These macros have javascript functions that will modify a html input tag to an interactive ajax box
with interaction to the lookup service that will let the user use autocomplete and search for lookup users and groups.

If you want to include an input box to let the user search and introduce a single user or a list of users, use the 
ucamlookup_users macro. You should pass as parameters to the macro the html input tag *id* and if you want to let the 
user select one or more users with the parameter *multiple*:

```python
    {% include 'ucamlookup_users.html' with input_tag_id="lookup_users" multiple=true %}
```

and you will also have to include the following macro in the head of your template to load the js and css files 
associated (these macros require jquery).

```python
    <script src="{% static 'jquery-1.11.1.min.js' %}"></script>
    {% include 'ucamlookup_headers.html' %}
```

And your input tag will be transform into an ajax box that allows the user to search for users using lookup either
using their username or their complete name. A list of crsids will be sent as the value of the input tag.

The same will work for lookup groups, just substitute user by group in the id and in the include.


## Admin interface

The admin interface is tunned to add managing options for the LookupGroup model. The *add* option will show the same
ajax-lookup-integrated-input as the template macros described above.

It also changes the add form for the user and it also shows an interactive ajax lookup-integrated input form when the
admin wants to add a new user to the app.

These input forms allow to search for name and crsid in the case of a new user and for name in the case of a lookup 
group.


## Available functions

The module also provides some useful functions to use in your app that do all the calls to the lookup service needed.

*get_group_ids_of_a_user_in_lookup(user)*: Returns the list of group ids of a user

*user_in_groups(user, lookup_groups)*: Check in the lookup webservice if the user is member of any of the groups in the 
LookupGroup list passed by parameter. Returns True if the user is in any of the groups or False otherwise

*def get_institutions(user=None)*: Returns the list of institutions using the lookup ucam service. The institutions of 
the user passed by parameters will be shown first in the list returned

*get_institution_name_by_id(institution_id, all_institutions=None)*: Returns the name of an institution by the id 
passed. If all_institutions is passed (the result from get_institutions) then the search is done locally using this 
list instead of a lookup call.

The last two methods can be used to add institutions to a model and show the name instead of the code in the admin 
interface 

```python
class MyModelAdmin(ModelAdmin):
    all_institutions = get_institutions()
    
    model = MyModel
    list_display = ('institution', )
    list_filter = ('institution_id', )

    def institution(self, obj):
        return get_institution_name_by_id(obj.institution_id, self.all_institutions)
        
    institution.admin_order_field = 'institution_id'
```