(volto-talk-listview-label)=

# Volto View Components: A Listing View for Talks

::::{sidebar} Volto chapter

:::{figure} _static/volto.svg
:alt: Volto Logo
:::

This chapter is about the React frontend Volto.

Solve the same tasks in Plone Classic in chapter {doc}`views_3`

:::{topic} Description
Create a view that shows a list of content
:::
::::

:::{sidebar} Get the code! ({doc}`More info <code>`)
Code for the beginning of this chapter:

```
git checkout talkview
```

Code for the end of this chapter:

```
git checkout talklist
```
:::

To be solved task in this part:

- Create a view that shows a list of talks

In this part you will:

- Create the view component
- Register a react view component for folderish content types
- Use an existing endpoint of Plone REST API to fetch content
- Display the fetched data

:::{only} not presentation
Volto has has a default view for content type `Document`.
The talk list should list all talks in this folderish page and also show information about the dates, the locations and the speakers. We will create an additonal view for the content type `Document`.
:::

## Register the view in Volto and Plone

Create a new file {file}`src/components/Views/TalkList.jsx`.

As a first step the file will hold a simple view component:

```jsx
import React from 'react';

const TalkListView = props => {
  return <div>I'm the TalkList component!</div>;
};
export default TalkListView;
```

As a convention we provide the view from {file}`src/components/index.js`.

```{code-block} jsx
:emphasize-lines: 2,4

import TalkView from './Views/Talk';
import TalkListView from './Views/TalkList';

export { TalkView, TalkListView };
```

Now register the new component as a view for folderish types in `src/config.js`.

```{code-block} jsx
:emphasize-lines: 1,13-16
:linenos: true

import { TalkListView, TalkView } from './components';

// All your imports required for the config here BEFORE this line
import '@plone/volto/config';

export default function applyConfig(config) {
  config.views = {
    ...config.views,
    contentTypesViews: {
      ...config.views.contentTypesViews,
      talk: TalkView,
    },
    layoutViews: {
      ...config.views.layoutViews,
      talklist_view: TalkListView,
    }
  };
  return config;
}
```

This extends the list of available views with the `talklist_view`.

To add a layout view you also have to add this new view in the `ZMI` of your `Plone`. Login to your Plone instance. Go to `portal_types` and select the `Document`-Type to add your new `talklist_view` to the `Available view methods`.

:::{figure} _static/add_talklistview_in_zmi.png
:alt: Add new View to content type Folder in the ZMI.

Add new View to content type Document in the ZMI.
:::

::::{only} not presentation

:::{warning}
This step is not in the final code for this chapter since it only changes the frontend, you need to do it manually for now.
It will be added in the next chapter where you change the backend-code.

The change would be in {file}`profiles/default/types/Document.xml`:

```{code-block} xml
:emphasize-lines: 5-7
:linenos: true

<?xml version="1.0"?>
<object name="Document" meta_type="Dexterity FTI" i18n:domain="plone"
    xmlns:i18n="http://xml.zope.org/namespaces/i18n">
  <property name="filter_content_types" purge="false">False</property>
  <property name="view_methods" purge="false">
    <element value="talklist_view"/>
  </property>
  <property name="behaviors" purge="false">
    <element value="plone.constraintypes"/>
  </property>
</object>
```
:::
::::

From now on you can select the new view for folder:

:::{figure} _static/talklistview_select.png
:::

Now we will improve this view step by step. We start working directly with the context of our talks folder. The context is part of the props of the view. To have a convenient access to the context we assign a variable `content` the value of `props.content`.

Via prop `content` we have access to title, description and other attributes

```{code-block} jsx
:emphasize-lines: 5
:linenos: true

import React from 'react';
import { Container } from 'semantic-ui-react';
import { Helmet } from '@plone/volto/helpers';

const TalkListView = ({content}) => {
  return (
    <Container className="view-wrapper">
      <Helmet title={content.title} />
      <article id="content">
        <header>
        <h1 className="documentFirstHeading">{content.title}</h1>
        {content.description && (
          <p className="documentDescription">{content.description}</p>
        )}
        </header>
      </article>
    </Container>
  )
};
export default TalkListView;
```

:::{only} not presentation
:::

(talklistview-search-endpoint-label)=

## Using the search endpoint

To get a list of all talks - no matter where they are in our site - we will use the `search endpoint` of the Plone REST API.
That is the equivalent of using a catalog search in classic Plone (see {ref}`views3-catalog-label`).

```{code-block} jsx
:emphasize-lines: 6-7,10-12,21-32

import React from 'react';
import { Container, Segment, Label, Image } from 'semantic-ui-react';
import { Helmet } from '@plone/volto/helpers';
import { Link } from 'react-router-dom';
import { flattenToAppURL } from '@plone/volto/helpers';
import { searchContent } from '@plone/volto/actions';
import { useDispatch, useSelector } from 'react-redux';

const TalkListView = ({ content }) => {
  const talks = useSelector(
    (state) => state.search.subrequests.conferencetalks?.items,
  );
  const dispatch = useDispatch();

  const color_mapping = {
    Beginner: 'green',
    Advanced: 'yellow',
    Professional: 'purple',
  };

  React.useEffect(() => {
    dispatch(
      searchContent(
        '/',
        {
          portal_type: ['talk'],
          fullobjects: true,
        },
        'conferencetalks',
      ),
    );
  }, [dispatch]);

  return (
    <Container className="view-wrapper">
      <Helmet title={content.title} />
      <article id="content">
        <header>
          <h1 className="documentFirstHeading">{content.title}</h1>
          {content.description && (
            <p className="documentDescription">{content.description}</p>
          )}
        </header>
        <section id="content-core">
          {talks &&
            talks.map(item => (
              <Segment padded>
                <h2>
                  <Link to={item['@id']} title={item['@type']}>
                    {item.type_of_talk.title || item.type_of_talk.token}:{' '}
                    {item.title}
                  </Link>
                </h2>
                {item.audience?.map(item => {
                  let audience = item.title || item.token;
                  let color = color_mapping[audience] || 'green';
                  return (
                    <Label key={audience} color={color}>
                      {audience}
                    </Label>
                  );
                })}
                {item.image && (
                  <Image
                    src={flattenToAppURL(item.image.scales.preview.download)}
                    size="small"
                    floated="right"
                    alt={content.image_caption}
                    avatar
                  />
                )}
                {item.description && <div>{item.description}</div>}
                <Link to={item['@id']} title={item['@type']}>
                  read more ...
                </Link>
              </Segment>
            ))}
        </section>
      </article>
    </Container>
  );
};

export default TalkListView;
```

::::{only} not presentation
We make use of the `useSelector` and `useDispatch` hooks from the react-redux library. They are used to subscribe our component to the store changes (`useSelector`) and for issuing Redux actions (`useDispatch`) from our components.

Afterwards we can define the new results with `const results = searchRequests.items;`, which will use the hooks and actions to receive a map of items.

The search itself will be defined in the `React.useEffect(() => {})`- section of the code and will contain all parameters for the search. In case of the talks listing view we search for all objects of type talk with `portal_type:['talk']` and force to fetch full objects with all information.

The items themselves won't change though, so the rest of the code will stay untouched.

Now you see all talks in the list no matter where they are located in the site.

:::{warning}
If you change the view in Volto you’ll also change the view in the backend (Plone). As long as the same view isn’t available in the backend too, the site will show an error!
:::
::::

## Search options

- The default representation for search results is a summary that contains only the most basic information like **title, review state, type, path and description**.
- With the option `fullobjects` all available field values are present in the fetched data.
- Another option is `metadata_fields`, which allows to get more attributes (selection of Plone catalog metadata columns) than the default search without a performance expensive fetch via option fullobjects.

Possible **sort criteria** are indices of the Plone catalog.

:::{seealso}
- Plone REST API documentation <https://plonerestapi.readthedocs.io/en/latest/searching.html>
- Plone documentation about searching and indexing <http://docs.plone.org/develop/plone/searching_and_indexing/query.html>
:::

(volto-talk-listview-exercise-label)=

## Exercises

### Exercise 1

Modify the criteria in the search to sort the talks in the order of their modification date.

:::{admonition} Solution
:class: toggle

```{code-block} python
:linenos: true

React.useEffect(() => {
  dispatch(
    searchContent('/', {
      portal_type: ['talk'],
      sort_on: 'modified',
      fullobjects: true,
    },
    'conferencetalks',
  ),
  );
}, [dispatch]);
```
:::

### Exercise 2

Change `TalkListView` to show the keynote speakers (name, biography and foto) and with a link to their keynote. Remember that you cannot search for a specific value in `type_of_talk` yet so you'll have to filter the results.

For bonus points create and register it as a separate view `Keynotes`

::::{admonition} Solution
:class: toggle

Write the view:

```{code-block} jsx
:emphasize-lines: 35-38
:linenos: true

import React from 'react';
import { Container, Segment, Image } from 'semantic-ui-react';
import { Helmet } from '@plone/volto/helpers';
import { Link } from 'react-router-dom';
import { flattenToAppURL } from '@plone/volto/helpers';
import { searchContent } from '@plone/volto/actions';
import { useDispatch, useSelector } from 'react-redux';

const TalkListView = ({ content }) => {
    const talks = useSelector(
      (state) => state.search.subrequests.conferencetalks?.items,
    );
  const dispatch = useDispatch();

  React.useEffect(() => {
    dispatch(
      searchContent('/', {
        portal_type: ['talk'],
        review_state: 'published',
        fullobjects: true,
      },
      'conferencetalks',
    ),
    );
  }, [dispatch]);

  return (
    <Container className="view-wrapper">
      <Helmet title={content.title} />
      <article id="content">
        <header>
          <h1 className="documentFirstHeading">Our Keynote Speakers</h1>
        </header>
        <section id="content-core">
          {talks &&
            talks.map(
              item =>
                item.type_of_talk.title === 'Keynote' && (
                  <Segment padded>
                    <h2>{item.speaker}</h2>
                    {item.image && (
                      <Image
                        src={flattenToAppURL(
                          item.image.scales.preview.download,
                        )}
                        size="medium"
                        centered
                        alt={item.speaker}
                      />
                    )}
                    {item.speaker_biography && (
                      <div
                        dangerouslySetInnerHTML={{
                          __html: item.speaker_biography.data,
                        }}
                      />
                    )}
                    <h3>
                      Keynote:{' '}
                      <Link to={item['@id']} title={item['@type']}>
                        {item.title}
                      </Link>
                    </h3>
                  </Segment>
                ),
            )}
        </section>
      </article>
    </Container>
  );
};
export default TalkListView;
```

:::{note}
- The query uses `review_state: 'published'`
- Filtering is done using `item.type_of_talk.title === 'Keynote' && (...` during the iteration.
:::

To regster it move the code to new {file}`frontend/src/components/Views/Keynotes.jsx` and rename it to `KeynotesView`:

```jsx
const KeynotesView = props => {
  [...]
}

export default KeynotesView;
```

Export it in {file}`frontend/src/components/index.js`:

```{code-block} jsx
:emphasize-lines: 3,5

import TalkView from './Views/Talk';
import TalkListView from './Views/TalkList';
import KeynotesView from './Views/Keynotes';

export { TalkView, TalkListView, KeynotesView };
```

Register the component as layout view for folderish types in `frontend/src/config.js`.

```{code-block} jsx
:emphasize-lines: 1,10

import { TalkListView, TalkView, KeynotesView } from './components';

[...]

config.views = {
  ...config.views,
  layoutViews: {
    ...config.views.layoutViews,
    talklist_view: TalkListView,
    keynotes_view: KeynotesView,
  },
  contentTypesViews: {
    ...config.views.contentTypesViews,
    talk: TalkView,
  },
};
```
::::
