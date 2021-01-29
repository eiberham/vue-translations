<template>
  <div id="home">
    <!-- <img class="logo" src="../assets/logo.png" alt="logo" /> -->

    <h1>{{ $t('views.home.helloWorld') }}</h1>

    <button class="button" v-if="!signedin" type="button" @click="auth">Sign In</button>
    <a v-else href="#" @click="signOut">Sign out</a> 

    <section v-if="signedin" class="events">
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
        signedin: false,
        events: []
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
      auth: async function(){
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

        this.signedin = true;

        this.getCalendarEvents();

      },
      signOut: function() {
        const auth2 = gapi.auth2.getAuthInstance();
        auth2.signOut().then(() => {
          this.signedin = false;
          console.log('User signed out.');
        });
      },
      getCalendarEvents: function() {
        gapi.client.calendar.events.list({
          'calendarId': 'primary',
          'timeMin': (new Date()).toISOString(),
          'showDeleted': false,
          'singleEvents': true,
          'maxResults': 10,
          'orderBy': 'startTime'
        }).then((response) => {
          const events = response.result.items;
          this.events = events;
        });
      
      }
    }
  }
</script>