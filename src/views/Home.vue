<template>
  <div id="home">
    <!-- <img class="logo" src="../assets/logo.png" alt="logo" /> -->

    <h1>{{ $t('views.home.helloWorld') }}</h1>

    <div v-if="!signedin" id="glogin" class="g-signin2" :data-onsuccess="onSignIn" :data-onfailure="onFailure" data-theme="dark"></div>
    <a v-else href="#" @click="signOut">Sign out</a> 

    <section v-if="signedin" class="events">
      <pre>
        {{ events }}
      </pre>
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
      gapi.load('client:auth2', () => {
            const auth2 = gapi.auth2.init({
                client_id: '611435266062-0oqhm1gf59fl3paeg5ru18fbkhcqn44d.apps.googleusercontent.com',
                cookiepolicy: 'single_host_origin',
                scope: 'profile email'
            });
            const element = document.getElementById('glogin');
            auth2.attachClickHandler(element, {}, this.onSignIn, this.onFailure);

            gapi.client.init({
              clientId: '611435266062-0oqhm1gf59fl3paeg5ru18fbkhcqn44d.apps.googleusercontent.com',
              discoveryDocs: ["https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest"],
              scope: 'https://www.googleapis.com/auth/calendar'
            });
      });
    },
    methods: {
      onSignIn: function(googleUser) {
        // Useful data for your client-side scripts:
        let id_token = googleUser.getAuthResponse().id_token;
        const profile = googleUser.getBasicProfile();

        console.log("id: "          + profile.getId()); // Don't send this directly to your server!
        console.log('full name: '   + profile.getName());
        console.log('given name: '  + profile.getGivenName());
        console.log('family name: ' + profile.getFamilyName());
        console.log("image url: "   + profile.getImageUrl());
        console.log("email: "       + profile.getEmail());

        // The id token you need to pass to your backend:
        id_token = googleUser.getAuthResponse().id_token;
        console.log("id token: " + id_token);
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
      onFailure: function(_error) {
        console.error(_error);
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