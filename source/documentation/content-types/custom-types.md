---
title: Custom Types
slug: content-types/custom-types

---

Creating custom content types with Sculpin is relatively easy. It mostly
involves configuring `sculpin_content_types` in `app/config/sculpin_kernel.yml`.
The end result is one or more data providers and optionally some taxonomy
related data providers and index generators.

The content types bundle uses some magic to derive some values automatically so
that you can configure less things. Everything can be overridden, though, so you
are not going to be stuck with some unfortunate decisions made by Sculpin's
default logic.


---

## Configuration

Content types are defined by configuring the `sculpin_content_types` section of
`app/config/sculpin_kernel.yml`. The top level keys in this node are considered
the type names. Each type can have additional configuration.

### Type Configuration Keys

For a given type name (the keys in the `sculpin_content_types` configuration),
the following keys are available:

 * **singular_name**:
   The singularized name for the type. Defaults to the result of calling a
   singularize function on the type name.
 * **type**:
   The type of content type, either **path** or **meta**. Essentially, this is
   how Sculpin determines *how* it will select the content type. Either based on
   path or on some meta information.
 * **path**:
   If `type: path`, the path that will be used to locate this type. Defaults to
   the singularized version of the type name with a `_` prepended. (so, for a
   content type named `talks`, the default path would be `_talks`)
 * **meta_key**:
   If `type: meta`, the meta key that will be used to locate this type. Defaults
   to the singularized version of the type name.
 * **meta**:
   If `type: meta`, the value of the meta key that will be used to locate this
   type. Defaults to the singularized name.
 * **publish_drafts**:
   Whether or not to publish drafts of this type. In the production environment
   this defaults to **false**. Otherwise, the default value is **true**.
 * **layout**:
   The default layout to use for this type. Defaults to the singularized name
   of the type.
 * **permalink**:
   The default permalink to use for this type.
 * **enabled**:
   Whether or not the content type should be enabled. Defaults to `true`.

### Frontmatter YAML Structures

The YAML frontmatter is parsed and injected into every page rendering
and is accessible as `page.KEY`.

Sculpin will read deep structures in YAML frontmatter. In your output file, use the dot notation to indicate that you want to descend into a structure. For example:

    ---
    layout: default
    something:
        here:
            very: deep
            also: deep
    ---

    {% verbatim %}We can reference `{{ page.something.here.also }} === deep`.{% endverbatim %}

---

## A Sample Content Type

The following is a sample content type called **projects**. This can be used as
a reference to see how everything fits together.


### Configuration

For demonstration purposes, we'll include all of the values even though we don't
really need to set all of them.

    sculpin_content_types:
        projects:
            type: path
            path: _projects
            singular_name: project
            layout: project
            enabled: true
            taxonomies:
                - tags

The same configurationc an be achieved by the following:

    sculpin_content_types:
        projects:

Magic is fun!


### Data Providers and Generators

Given the above configuration, a few data providers and a generator will be made
available:

 * **projects**:
   This is created based on the name of the type. It contains a sorted list of
   all of the sources that are discovered for the content type.
    * **next_project**:
      Each item in the **projects** collection will have a **next_project**
      meta data that contains either **null** or the next project in the
      collection.
    * **previous_project**:
      Each item in the **projects** collection will have a **previous_project**
      meta data that contains either **null** or the previous project in the
      collection.
 * **projects_tags**:
   This is a `Sculpin\Contrib\Taxonomy\ProxySourceTaxonomyDataProvider` that
   contains a mapping from tags to projects.
 * **projects_tag_index**:
   This is a `Sculpin\Contrib\Taxonomy\ProxySourceTaxonomyIndexGenerator` that
   will create a page for each tag.
    * **tag**:
      Each generated tag index will have a piece of meta data named **tag** that
      will contain the name of the tag for which the index is being generated.
    * **tag_projects**:
      Each generated tag index will have a piece of meta data named
      **tag_projects** that will be the collection of all projects with this tag.

### Sample Projects Index Page

The following is a quick sample of a template that can be used to list all
projects.

    ---
    use: [projects]
    ---
    {% verbatim %}<ul>
        {% for project in data.projects %}
            <li><a href="{{ project.url }}">{{ project.title }}</a></li>
        {% endfor %}
    </ul>{% endverbatim %}

### Sample Project Tags Index Page

The following is a quick sample of a template that can be used to list all
tags that have been used to tag projects.

    ---
    use: [projects_tags]
    ---
    {% verbatim %}<ul>
        {% for tag,projects in data.projects_tags %}
            <li><a href="{{ site.url }}/projects/tags/{{ tag|url_encode(true) }}">{{ tag }}</a></li>
        {% endfor %}
    </ul>{% endverbatim %}

### Sample Project Tags Projects Index Page

The following is a quick sample of a template that leverages the tag index
generator to create a new page for every tag and list its projects.

    ---
    generator: projects_tag_index
    ---
    <h1>{{ page.tag }}</h1>
    {% verbatim %}<ul>
        {% for project in page.tag_projects %}
            <li><a href="{{ project.url }}">{{ project.title }}</a></li>
        {% endfor %}
    </ul>{% endverbatim %}

