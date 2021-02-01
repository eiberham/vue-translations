<template>
  <div id="home">
    <!-- <img class="logo" src="../assets/logo.png" alt="logo" /> -->

    <h1>{{ $t('views.home.helloWorld') }}</h1>
    
    <button class="button" v-if="!signedIn" type="button" @click="signIn">Sign In</button>
    <a v-else href="#" @click="signOut">Sign out</a> 

    <section v-if="signedIn" class="events">
      <ul>
        <li v-for="event in events" :key="event.id" style="list-style: none; margin: 8px 0; background: #f2f2f2; border-radius: 8px; padding: 1rem;">
          <div>
            <strong>{{ event.summary }}</strong>
            <p>{{ event.description }}</p>
            <span>from: {{ new Date(event.start.dateTime).toISOString() }} to {{ new Date(event.end.dateTime).toISOString() }}</span>
          </div>
        </li>
      </ul>
    </section>
  </div>
</template>

<script>
  export default {
    components: {},
    data: function(){
      return {
        signedIn: false,
        events: [],
        syncInterval: 1000 * 30,
      }
    },
    mounted: function() {
      this.init();
    },
    methods: {
      /**
       * Initializes the google api client library.
       */
      init: function() {
        gapi.load('client', () => {
            gapi.client.init({
              apiKey: 'AIzaSyCyO0fSFRTQ62NxiLoAPBcbUpvLE5XWToA',
              clientId: '611435266062-0oqhm1gf59fl3paeg5ru18fbkhcqn44d.apps.googleusercontent.com',
              discoveryDocs: ["https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest"],
              scope: 'https://www.googleapis.com/auth/calendar https://www.googleapis.com/auth/calendar.events'
            });

            gapi.client.load('calendar', 'v3', () => console.log('calendar ready'));
        });
      },
      /**
       * Signs in into google account.
       */
      signIn: async function() {
        const googleAuth = gapi.auth2.getAuthInstance();
        const googleUser = await googleAuth.signIn();

        const token = googleUser.getAuthResponse().id_token;
        if (token) {
          this.signedIn = true;

          // The current date in simplified extended iso format
          const date = new Date();
          const timeMin = date.toISOString();
          // Plus one day
          const timeMax = new Date(date.setDate(date.getDate() + 2)).toISOString();

          this.getCalendarEventsByRange(timeMin, timeMax);
        }
      },
      /**
       * Signs out from google account.
       * @returns {void}
       */
      signOut: function() {
        const auth2 = gapi.auth2.getAuthInstance();
        auth2.signOut().then(() => {
          this.signedIn = false;
          console.log('User signed out.');
        });
      },
      /**
       * Makes "incremental synchronization" of calendar data.
       * 
       * @params {object} eventParams - 
       * @params {number} syncInterval - the synchronization interval.
       * @returns {void}
       * @see {@link https://developers.google.com/calendar/v3/sync} for further information.
       */
      getSyncEvents: async function(eventParams, syncInterval) {
        // Performs initial full sync.
        const response = await this.getCalendarEvents(eventParams);
        this.events = response.result.items;
        if (response.result.nextSyncToken) this.setSyncToken(response.result.nextSyncToken);

        // Performs incremental sync.
        const getIncrementalSync = async () => {
          try {
            const syncToken = this.getSyncToken();
            const response = await this.getCalendarEvents({
              ...eventParams,
              ...(syncToken) && { nextSyncToken: syncToken },
            });
            if (response.result.nextSyncToken) this.setSyncToken(response.result.nextSyncToken);
            this.events = response.result.items;
          } catch(error) {
            console.log(error);
            clearInterval(intervalId);
            this.getSyncEvents(eventParams, syncInterval);
          } 
        };

        const intervalId = setInterval(getIncrementalSync, syncInterval);
      },
      /**
       * Retrieves calendar events in the specified range
       * 
       * @params {object} params the event params
       * @returns {Promise} promise containing calendar events
       */
      getCalendarEvents: function(params) {
        return gapi.client.calendar.events.list({
          ...params
        });
      },
      /**
       * Gets sync token from local storage
       * 
       * @returns {string} sync token stored previously
       */
      getSyncToken: function(){
        const syncToken = window.localStorage.getItem('nextSyncToken');
        return syncToken;
      },
      /**
       * Persists sync token in local storage
       * 
       * @params {string} sync token to be persisted
       * @returns {void}
       */
      setSyncToken: function(syncToken){
        if (!syncToken) return;
        window.localStorage.setItem('nextSyncToken', syncToken);
      },
      /**
       * Get google calendar events by range
       * 
       * @params {number} timeMin - starting point
       * @params {number} timeMax - ending point
       * @returns {void}
       */
      getCalendarEventsByRange: function(timeMin = null, timeMax = null){
        this.getSyncEvents({
          calendarId: 'primary',
          timeMin: timeMin || (new Date()).toISOString(),
          ...(timeMax) && { timeMax },
          showDeleted: false,
          singleEvents: true,
          maxResults: 10,
        }, this.syncInterval);

        return;
      }
    }
  }
</script>