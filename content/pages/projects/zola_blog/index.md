+++
title = "Comittee of Zola (Jekyll -> Zola SSG conversion)"
date = 2024-03-15
[extra]
group = ""
+++

This is the site you're browsing right now - it's a basic port of the Comittee of Zero website <https://sonome.dareno.me/> from [Jekyll](https://jekyllrb.com/) to [Zola](https://www.getzola.org/). In this post I'll briefly go over how I chose to adapt their Jekyll blog format to Zola. 

Comittee of Zero is a visual novel port/patch/translation team that works on 5pb./MAGES.’s Science Adventure (SciADV) series of games (most notably STEINS;GATE), and their blog has a no-frills layout that's [simple and clean](https://www.youtube.com/watch?v=B1nDzB1P8GM)... and open source! For future reference, you can find the Comittee of Zero repo at <https://github.com/CommitteeOfZero/committeeofzero.github.io> and the repo for this site at <https://github.com/mgardos01/mgardos01.github.io>. 

I saw that they used Jekyll to build their blog, and figured that adapting their clean site layout from scratch would be a good way to learn some trendy flavor of SSG that I'd read about in some hackernews discussion. 

## Why Zola? 
- Super fast builds (the main selling point)
- Single binary 
- Templating language (Tera) is super similar to Jekyll's Liquid templates
- RSS feed support out of the box
- I tried Hugo first and came across some syntax that was so much uglier than anything in Jekyll so I immediately nope'd out and chose the second one I remembered 

The [quick overview](https://www.getzola.org/documentation/getting-started/overview/#first-steps-with-zola) in the Zola docs was also super easy to follow. 

<!-- Quick disclaimer: I gutted most of the metadata out of the original files -->

## Project Structure 
I'll go over how I translated the structure of the Jekyll project over first before going over any substantial changes I made to the layout logic.

### HTML Templates 
Jekyll: 
```bash
─── index.html
_includes/
├── head.html
├── header.html
└── thumb.html
_layouts/
├── default.html
└── page.html
```

Zola: 
```bash
templates/
├── default.html
├── index.html
├── macros.html < macro::{head, header}
├── page.html
├── section.html
└── shortcodes
    └── thumb.html
```

In Jekyll, the ```_layouts``` directory contains the templates that wrap markdown content, which is slotted in with a ```{{ content }}``` tag. The ```_includes``` directory contains partial HTML components that can be mixed and matched by your layouts and included in markdown files. 

In Zola, every template lives in the ```templates``` directory. Partials that are meant to be included in other templates like ```head``` and ```header``` elements are recommended to be written as Tera macros which are imported and namespaced. Partials that are meant to be included primarily in markdown files are recommended to be written as shortcodes and placed in the ```shortcodes``` directory (which exposes their use to every markdown file automatically without the need to add an import statement). 

In the Jekyll project, 
- ```default.html``` serves as the base template
- it slots in  ```head.html``` for metadata and  ```header.html``` for the navbar
- ```{{ content }}``` is slotted with the paginated list of blogposts from ```index.html``` or a single page of content from ```page.html```. 

In the Zola project, 
- ```default.html``` serves as the base template
- it slots in  ```macro::head()``` for metadata and  ```macro::header()``` for the navbar
- ```{{ content }}``` is slotted with the paginated list of blogposts from ```index.html``` or a single page of content from ```{page, section}.html```. ```section.html``` is only for the project listing - more on that later.   

### Sass 
Jekyll: 
```bash
_sass/
├── skeleton
│   ├── _loader.scss
│   └── _vars.scss
└── vendor
    ├── normalize-scss < git submodule
    └── skeleton-sass < git submodule
_assets/
└── main.scss
```

Zola: 
```bash
node_modules/
├── normalize-scss
└── skeleton-sass-official
sass/
├── _config.scss
├── _loader.scss
├── _vars.scss
└── skeleton.scss
```

In Jekyll, the ```_sass``` directory only contains sass partials which are imported by ```main.scss``` from the ```assets``` directory. 

Zola processes any files with the ```sass``` or ```scss``` extension in the ```sass``` directory and places the processed output into a ```css``` file with the same directory structure and base name into the ```public``` directory - the root of a built Zola project. 

### Assets 

I didn't sweat this one too much - all the image uploads in the original project lived in an  ```uploads``` directory and static assets (logos, svgs, etc.) in an ```assets``` directory. 

Zola comes with something they call "Asset colocation" - All non-Markdown files you add in a page directory are copied alongside the generated page when the site is built, which allows a relative path to be used access them, so I eschewed the ```uploads``` directory altogether, anticipating that I won't be reusing images in multiple blog posts too often. If I did want to, Zola's ```static``` directory is intended for any kind of static file - all the files/directories in this directory are be copied as-is to the output directory on build, so something like ```static/uploads``` would work (I use ```static/assets``` to hold the icons present on the navbar)

### Posts 

I didn't look at the way posts were routed/organized in Jekyll before this but I might as well include it. I'm using ```+++``` to denote information that's found in the front-matter (like the header) of each markdown file. 

Jekyll
```bash
─── about.md
─── projects.md
_posts/
├── example-blogpost-name.md
└── example-blogpost-but-this-is-also-a-project.md +++ permalink=project/somethingsomething
projects/
└── example-project-name.md +++ permalink=project/something
```

Zola
```bash
content/
├── blog/
│   ├── example-blogpost-name/
│   │   └── index.md
│   └── _index.md +++ transparent=true
├── pages/
│   ├── about/
│   │   └── _index.md
│   └── projects/
│       ├── example-project-name/
│       │   └── index.md
│       └── _index.md
└── _index.md +++ paginate_by=5
```

Without going into much detail, Zola has "pages" and "sections" organized in the ```content``` directory. Directories with a ```_index.md``` file are sections, and directories with a ```index.md``` file are pages. Sections can paginate the pages that are under them automatically, and can paginate the pages that are under their subsections if the subsection is "transparent". You can paginate posts on the homepage by putting every post in its own transparent section, which I named ```blog```, and every other page in an opaque section I named ```pages```.

See [this](https://www.getzola.org/documentation/content/overview/) for more details about section routing.
## What's Different 

### No "active_tab" variable in frontmatter
Every rendered website checks whether or not it's a "page" or a "section" before rendering the navbar. If it's a ```page```, it sets the ```active_tab``` equal to the title of its parent section. If it's a ```section```, it sets the ```active_tab``` equal to its title. The 
```c
{% import "macros.html" as macros %}
...
    <div class="two columns" id="menu-column">
        {% if page %}
            {% set active_tab = get_section(path = page.ancestors | last) | get(key="title") %}
        {% else %}
            {% set active_tab = section.title %}
        {% endif %}
        {{ macros::header(config=config, active_tab=active_tab) }}
    </div>
...
```

And some other stuff too I'm so tired I'll update this post later ok you get the gist I guess!
