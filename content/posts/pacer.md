---
title: "Pacer: MERN Stack Social Platform"
date: 2023-12-06T14:50:11-05:00
slug: pacer
type: posts
draft: false
---
<!-- ! Post Header -->
Full-stack MERN platform for finding exercise partners for your favorite sports
<div class="stack-logos">
  <img alt="MongoDB" class="tech-logo" src="https://img.shields.io/badge/-MongoDB-13aa52?style=flat-square&logo=mongodb&logoColor=white" />
  <img alt="Express" src="https://img.shields.io/badge/-Express-ffffff?style=flat-square&logo=express&logoColor=black" />
  <img alt="React" src="https://img.shields.io/badge/-React-45b8d8?style=flat-square&logo=react&logoColor=white" />
  <img alt="Redux" src="https://img.shields.io/badge/-Redux-764ABC?style=flat-square&logo=redux&logoColor=white" />
  <img alt="Nodejs" src="https://img.shields.io/badge/-Node.js-43853d?style=flat-square&logo=Node.js&logoColor=white" />
</div>
<!-- ! "Read more..." -->
<!--more-->
<br>

## Table of Contents
- [Table of Contents](#table-of-contents)
- [Idea](#idea)
- [Features](#features)
- [Plan](#plan)
- [Tech Stack](#tech-stack)
- [Notable Highlights](#notable-highlights)
- [Implementation](#implementation)
  - [Data Architecture with Mongoose \& MongoDB](#data-architecture-with-mongoose--mongodb)
- [Google Maps API Integration](#google-maps-api-integration)
  - [Integrated three Google Maps API endpoints:](#integrated-three-google-maps-api-endpoints)
- [Express.js Routing](#expressjs-routing)
- [React.js Components](#reactjs-components)
- [State Management with Redux](#state-management-with-redux)
- [Challenges](#challenges)
- [Team](#team)
  - [Gary Jiang](#gary-jiang)
  - [Jason Jun](#jason-jun)
  - [Francis Cawog](#francis-cawog)
  - [Robert Lee](#robert-lee)

## Idea
*"I hate running alone but my friends run way too fast. I wish I had my own personal Pacer..."*
<br><br>
In the world of running *pacers* play a crucial role by setting a consistent speed to help runners achieve their target times in a race.
<br><br>
The *Pacer* platform helps you connect with people playing your favorite sport at similar performance levels so you can train, bond, and celebrate together.
<br><br>
👉
[Live Demo](https://pacer-65mk.onrender.com/)
👈

<br>

## Features
1. User Auth
2. Events (CRUD) w/ Interactive Google Map
3. Google Maps API Integration (Dynamic Map, Static Map, & Places API)
4. User Profiles
5. Discover Events Page
6. Event Social Stats
7. Event Comments (CRUD)

<br>

## Plan
1. Audience Value Props
![Audience-Value-Props](https://github.com/garysbot/pacer/raw/main/frontend/src/readme-imgs/audience-value-props.png)
<br>

1. App Architecture & Data Flow
![Data-Architecture-Flow](https://github.com/garysbot/pacer/raw/main/frontend/src/readme-imgs/data-architecture.png)

1. Design
![Design-1-Discover](https://github.com/garysbot/pacer/raw/main/frontend/src/readme-imgs/design-1-discover.png)
![Design-2-EventShow](https://github.com/garysbot/pacer/raw/main/frontend/src/readme-imgs/design-2-event-show.png)

<br>
<br>

## Tech Stack
- MongoDB
- ExpressJS
- React.js
- Node.js
<br>

## Notable Highlights
![User-Auth](https://github.com/garysbot/pacer/raw/main/frontend/src/icons/user-auth-login.gif)
![User-Auth-Sign-Up](https://github.com/garysbot/pacer/raw/main/frontend/src/icons/pacer-sign-up-modal.gif)
<br>*User Authentication Modals*<br>


![Sports-Ranking](https://github.com/garysbot/pacer/raw/main/frontend/src/icons/sports-feature.gif)
<br>*Event Filter*<br>


![Google-Map](https://github.com/garysbot/pacer/raw/main/frontend/src/icons/google-map.gif)
<br>*Google Maps API Integration*<br>
<br>


## Implementation
###  Data Architecture with Mongoose & MongoDB

Mongoose Models: `Events`, `Users`, `Comments`
```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const commentSchema = new Schema({
  owner: {
    type: Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  event: {
    type: Schema.Types.ObjectId,
    ref: 'Event',
    required: true
  },
  body: {
    type: String,
    required: true
  }
}, {
  timestamps: true
});

module.exports = mongoose.model('Comment', commentSchema);

```
<br>

Validations to ensure data integrity:
```javascript
const validateEventInput = [
    check('owner')
        .exists({ checkFalsy: true })
        .withMessage('User must be logged in to create an event'),
    check('eventName')
        .exists({ checkFalsy: true })
        .isLength({ min: 5, max: 100 })
        .withMessage('Event must have a name between 5 and 100 characters'),
    check('description')
        .exists({ checkFalsy: true })
        .isLength({ min: 5, max: 1000 })
        .withMessage('Event must have a description between 5 and 1000 characters'),
    check('locationName')
        .exists({ checkFalsy: true })
        .withMessage('Event location must have a name'),
        ...
```
<br>

## Google Maps API Integration

### Integrated three Google Maps API endpoints:
1. Dynamic interactive map on `Events` page.
2. Static map preview on `Discover` page.
3. Places API autocomplete search results when selecting an `Event` location.

![Places-Autocomplete](https://github.com/garysbot/pacer/raw/main/frontend/src/icons/places-autocomplete.gif)
<br>

```javascript
<GoogleMap
  mapContainerStyle={mapStyles}
  zoom={14}
  center={{
    lat: selectedEvent?.latitude,
    lng: selectedEvent?.longitude,
  }}
>
  <Marker 
    position={{
    lat: selectedEvent?.latitude,
    lng: selectedEvent?.longitude,
    }}
    onClick={handleMarkerClick}
  />
  {infoWindowVisible && (
    <InfoWindow
      position={{
        lat: selectedEvent?.latitude,
        lng: selectedEvent?.longitude,
      }}
      onCloseClick={() => setInfoWindowVisible(false)}
    >
      <div>
        <p>Location: {selectedEvent?.locationName}</p>
        <p>Time of Event: {formattedTime}</p>
      </div>
    </InfoWindow>
  )}
</GoogleMap>
```
<br>

## Express.js Routing
- Three RESTful Express routes for `Users`, `Events`, `Comments`
- Full CRUD for `Events` and `Comments`

```javascript
    
router.post('/', requireUser, validateCommentInput, async (req, res, next) => {
  try {
    const newComment = new Comment({
      owner: req.body.owner,
      event: req.body.event,
      body: req.body.body
    });

    let comment = await newComment.save();

    comment = await Comment.findById(comment._id)
      .populate('owner', '_id firstName lastName profilePhotoUrl')
      .populate('event', '_id eventName locationName dateTime difficulty eventType maxGroupSize longitude latitude');

    return res.json(comment);
  } catch (err) {
    next(err);
  }
}); 
```
<br>

## React.js Components
- `Discover` Page:
  - Primary user interface with comprehensive `Event` filtering feature.
  - Displays all `Events` available for `Users`.
  - `Event POST` feature to create new `Events`.
  - Session and `User` validation required to create new `Events`.
  ![Discover-Page](https://github.com/garysbot/pacer/raw/main/frontend/src/icons/discover.gif)

- `Event` Page 
  - Displays `Event` details from Redux state.
  - Attending & Interested buttons with integration to update Redux and MongoDB.
  - `Comments POST` feature to create new `Comments`.
![Event-Page](https://github.com/garysbot/pacer/raw/main/frontend/src/icons/events.gif)
<br>

## State Management with Redux
- Redux implemented for state management enhancing UI load times and reducing unnecessary API calls.
- Redux slices of state for `Users`, `Events`, `Comments`.
- `Events` reducer:
```javascript
const eventsReducer = (state = { all: {}, user: {}, new: undefined }, action) => {
  switch (action.type) {
    case RECEIVE_EVENTS:
      const allEventsArray = action.events;

      const allEventsObject = allEventsArray.reduce((accumulator, event) => {
        accumulator[event._id] = event;
        return accumulator;
      }, {});

      return { ...state, all: allEventsObject, new: undefined };
    case RECEIVE_NEW_EVENT:
      return { ...state, all: { ...state.all, [action.event._id]: action.event }, new: undefined };
    case UPDATE_EVENT:
      return { ...state, all: { ...state.all, [action.event._id]: action.event }, new: undefined };
    case DELETE_EVENT:
      const { [action.eventId]: deletedEvent, ...rest } = state.all;
      return { ...state, all: rest, new: undefined };
    case RECEIVE_EVENT:
      return { ...state, all: { ...state.all, [action.event._id]: action.event }, new: undefined };
    case RECEIVE_USER_LOGOUT:
      return { ...state, user: {}, new: undefined }
    default:
      return state;
  }
};
```

<br><br>

## Challenges
1. Three days to learn the MERN stack
    - Team was well-versed with React and Node.js however Mongoose, MongoDB, and Express were new technologies for the team.
    - The minimalism of Mongoose, and Express created a forgiving and flexible application architecture.
    - Primary learning challenges were Express for backend routes due to it's flexible nature vs. the team's more familiar Rails *convention over configuration* paradigm.

2. Google Maps and parsing location data
    - Implementing three different Google Maps API end points proved to be challenging.
    - (1) Dynamic Interactive Map, (2) Static map image, and (3) Places API search autocomplete suggestions
    - Each React component required different location data parsing logic to ensure UI loading efficiency across the user experience.

3. Handling bugs
    - Debugging helped us understand the stack better via tracing bugs throughout the application.
    - Developed a three-tier approach to assess bugs efficiently across a full-stack application:
        (1) Backend logs
        (2) Frontend console logs
        (3) Middleware logic assessment

<br><br>

## Team
### Gary Jiang
*Team & Frontend Lead*<br>
[Github](https://github.com/garysbot)<br>
[Website](https://imgaryjiang.com)<br>
[LinkedIn](https://linkedin.com/in/garyjiang)<br>

### Jason Jun
*Flex Lead*<br>
[Github](https://github.com/junjason)<br>
[Website](https://www.junjason.com)<br>
[LinkedIn](https://www.linkedin.com/in/jason-jun-0a7576237/)<br>

### Francis Cawog
*Backend Lead*<br>
[Github](https://github.com/garysbot)<br>
[Website](https://imgaryjiang.com)<br>
[LinkedIn](https://linkedin.com/in/garyjiang)<br>

### Robert Lee
*Flex*<br>
[Github](https://github.com/garysbot)<br>
[Website](https://imgaryjiang.com)<br>
[LinkedIn](https://linkedin.com/in/garyjiang)<br>

