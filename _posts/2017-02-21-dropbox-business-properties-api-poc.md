---
layout: post
title: 'Dropbox Business API: Properties API Proof of Concept'
tags:
 - dropbox
 - dropbox business
 - api
 - python
---

I wrote sample script of Properties API of Dropbox Business in Python.

Note: Properties APIs are alpha release (as of Feb 2017). Specification of APIs may change without notice.

# Properties API

Dropbox API and Dropbox Business API provide store properties of files or folders.
These properties are not displayed on desktop, mobile and web.
Properties APIs are designed for application which integrate with Dropbox Business.
Here are possible use cases.

* Store security policies for each files/folders.
* Store approval status of documents.
* Store document ID which issued from other CMS.

## Properties template

Before storing properties of files/folders. You need to create Properties template in Dropbox Business team. To perform properties template operation, application must have "Team member file access" token. See more detail about auth type at [Access Type](https://www.dropbox.com/developers/documentation/http/teams#access-types).

Properties template can have multiple fields. Field can have single string value.

Here are APIs for Properties template. No method for delete template. Applications should maintain properties template id. Because you can create same name properties template. Template name is just for human friendly purpose.

* [/team/properties/template/add](https://www.dropbox.com/developers/documentation/http/teams#team-properties-template-add)
* [/team/properties/template/get](https://www.dropbox.com/developers/documentation/http/teams#team-properties-template-get)
* [/team/properties/template/list](https://www.dropbox.com/developers/documentation/http/teams#team-properties-template-list)
* [/team/properties/template/update](https://www.dropbox.com/developers/documentation/http/teams#team-properties-template-update)

Here is example of list existing templates:

```python
templates = client.team_properties_template_list()

for t in templates.template_ids:
    template = client.team_properties_template_get(t)
    print "Template Id: %s" % t
    print "Template Name: %s" % template.name
    print "Description: %s" % template.description
    for f in template.fields:
        print "Field[%s] Description[%s]" % (f.name, f.description)
```

## Store/retrieve properties of files/folders

To retrieve properties of file/folder, use [/files/alpha/get_metadata](https://www.dropbox.com/developers/documentation/http/documentation#files-alpha-get_metadata). For retrieve properties, specify template id in `include_property_templates` attribute. 

To store properties of file/folder, use [/files/properties/add](https://www.dropbox.com/developers/documentation/http/documentation#files-properties-add) or [/files/properties/overwrite](https://www.dropbox.com/developers/documentation/http/documentation#files-properties-overwrite).

# Code sample

Here is complete code sample of use case of properties APIs.
This sample expect to store security policy and levels for each files. This sample code using latest [Dropbox Python SDK](https://github.com/dropbox/dropbox-sdk-python).

```python
import dropbox
from dropbox.files import PropertyGroupUpdate
from dropbox.properties import PropertyFieldTemplate, PropertyType, PropertyField, PropertyGroup

# Dropbox Business Team file access token
DROPBOX_TEAM_FILE = ''


def list_more_files(client, cursor):
    chunk = client.files_list_folder_continue(client, cursor)
    if chunk.has_more:
        return chunk.entries + list_more_files(client, chunk.cursor)
    else:
        return chunk.entries


def list_files(client, path):
    # Set recursive=False because files under shared folders are not listed.
    # call traverse_files for extract files recursively
    chunk = client.files_list_folder(path, recursive=False)
    if chunk.has_more:
        return traverse_files(client, chunk.entries + list_more_files(client, chunk.cursor))
    else:
        return traverse_files(client, chunk.entries)


def traverse_files(client, entries):
    all = []
    for f in entries:
        all.append(f)
        if isinstance(f, dropbox.files.FolderMetadata):
            all += list_files(client, f.path_lower)

    return all


def list_more_members(client, cursor):
    chunk = client.team_members_list_continue(cursor)
    if chunk.has_more:
        return chunk.members + list_more_members(client, chunk.cursor)
    else:
        return chunk.members


def list_members(client):
    chunk = client.team_members_list()
    if chunk.has_more:
        return chunk.members + list_more_members(client, chunk.cursor)
    else:
        return chunk.members


def show_properties_templates(client):
    templates = client.team_properties_template_list()

    for t in templates.template_ids:
        template = client.team_properties_template_get(t)
        print "Template Id: %s" % t
        print "Template Name: %s" % template.name
        print "Description: %s" % template.description
        for f in template.fields:
            print "Field[%s] Description[%s]" % (f.name, f.description)


def find_properties_template_id_by_name(client, name):
    templates = client.team_properties_template_list()
    for t in templates.template_ids:
        template = client.team_properties_template_get(t)
        if template.name == name:
            return t

    return None


def audit_file(client, security_policy_template_id, file):
    print "Auditing file: %s" % file.path_display

    meta = client.files_alpha_get_metadata(file.path_lower, include_property_templates=[security_policy_template_id])
    if isinstance(meta, dropbox.files.FileMetadata) and meta.property_groups is not None:
        for p in meta.property_groups:
            if p.template_id == security_policy_template_id:
                for field in p.fields:
                    print "File[%s] Security Policy: %s = %s" % (file.path_display, field.name, field.value)

    # Mark as Confidential for every '.pdf'
    if file.path_lower.endswith('.pdf'):
        print "Updating security policy: %s : template_id=%s" % (file.path_display, security_policy_template_id)
        level_field = PropertyField('Level', 'Confidential')
        prop_group = PropertyGroup(security_policy_template_id, [level_field])
        client.files_properties_overwrite(file.path_lower, [prop_group])


def audit_member(client, security_policy_template_id, member):
    print "Auditing files of member: %s" % member.profile.email

    files = list_files(client, "")
    for f in files:
        if isinstance(f, dropbox.files.FileMetadata):
            audit_file(client, security_policy_template_id, f)


if __name__ == '__main__':
    client_team = dropbox.DropboxTeam(DROPBOX_TEAM_FILE)

    # List existing properties template
    show_properties_templates(client_team)

    # Lookup template named 'Security Policy'
    tmpl_name = 'Security Policy'
    tmpl_desc = 'These properties describe how confidential this file is.'
    tmpl_field_level_name = 'Level'
    tmpl_field_level_desc = 'Level can be Confidential, Public or Internal.'

    security_policy_template_id = find_properties_template_id_by_name(client_team, tmpl_name)

    # Add template if not exist
    if security_policy_template_id is None:
        fields = [
            PropertyFieldTemplate(tmpl_field_level_name, tmpl_field_level_desc, PropertyType.string)
        ]
        security_policy_template_id = client_team.team_properties_template_add(tmpl_name, tmpl_desc, fields)
        print security_policy_template_id

    # Audit member files
    members = list_members(client_team)
    for m in members:
        # Create client as user
        c = client_team.as_user(m.profile.team_member_id)
        audit_member(c, security_policy_template_id, m)
```