<template>
  <div id="home">
    <!-- <img class="logo" src="../assets/logo.png" alt="logo" /> -->

    <h1>{{ $t('views.home.helloWorld') }}</h1>
    <!-- <button type="button" @click="signOut">Sign out</button> -->
    <!-- <button v-if="!signedin" class="button" type="button" @click="authorize">Authorize</button> -->
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
    mounted: function(){
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
    methods: {
      authorize: function(){
        gapi.auth.authorize(
          {
            'client_id': '611435266062-0oqhm1gf59fl3paeg5ru18fbkhcqn44d.apps.googleusercontent.com',
            'scope': 'https://www.googleapis.com/auth/calendar https://www.googleapis.com/auth/calendar.events',
            'immediate': true
          }, (done) => {
            console.log("authed: ", done)
            gapi.client.load('calendar', 'v3', () => console.log('calendar ready'));
          });
      },
      /**
       * Signs in into google account.
       */
      signIn: async function (){
        const googleAuth = gapi.auth2.getAuthInstance();
        const googleUser = await googleAuth.signIn();

        const profile = googleUser.getBasicProfile();

        console.log("id: "          + profile.getId());
        console.log('full name: '   + profile.getName());
        console.log('given name: '  + profile.getGivenName());
        console.log('family name: ' + profile.getFamilyName());
        console.log("image url: "   + profile.getImageUrl());
        console.log("email: "       + profile.getEmail());

        const token = googleUser.getAuthResponse().id_token;
        console.log("token: ", token);

        this.signedIn = true;

        const eventParams = {
          calendarId: 'primary',
          timeMin: (new Date()).toISOString(),
          // timeMax: (new Date()).toISOString(),
          showDeleted: false,
          singleEvents: true,
          maxResults: 10,
        };

        this.getSyncEvents(eventParams, this.syncInterval); //(this.syncInterval);

      },
      /**
       * Signs out from google account.
       */
      signOut: function() {
        const auth2 = gapi.auth2.getAuthInstance();
        auth2.signOut().then(() => {
          this.signedIn = false;
          console.log('User signed out.');
        });
      },
      /**
       * 
       * 
       */
      getSyncEvents: async function(eventParams, syncInterval) {
        // debugger;
        const response = await this.getCalendarEvents(eventParams);
        this.events = response.result.items;
        if (response.result.nextSyncToken) this.setSyncToken(response.result.nextSyncToken);

        // The result of this list request contain only entries that have changed.
        const getIncrementalSync = async () => {
          try {
            const syncToken = this.getSyncToken();
            const response = await this.getCalendarEvents({
              ...(syncToken) && { nextSyncToken: syncToken },
              singleEvents: true,
              maxResults: 10,
            });
            if (response.result.nextSyncToken) this.setSyncToken(response.result.nextSyncToken);
          } catch(error) {
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
       */
      getCalendarEvents: function(params) {
        console.log("params: ", params);
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
       */
      setSyncToken: function(syncToken){
        if (!syncToken) return;
        window.localStorage.setItem('nextSyncToken', syncToken);
      },
    }
  }
</script>