<template>
  <v-card class="container mx-auto mt-5">
    <v-card-title>
      <div class="flex-container">
        <h1 class="display-1">My Exercises</h1>
        <v-btn :to="{ name: 'generate-workout' }" color="primary"
          >Generate Workout</v-btn
        >
      </div>
    </v-card-title>
    <v-card-text>
      <v-expansion-panels multiple @change="fixParticles">
        <v-expansion-panel v-for="(e, index) in exercises" :key="index">
          <div :class="completedExercises[index] ? 'complete' : 'incomplete'">
            <v-expansion-panel-header expand-icon="mdi-menu-down">
              <b>{{ e.exerciseName }}</b>
            </v-expansion-panel-header>
            <v-expansion-panel-content>
              <p>
                {{ e.description }}
              </p>
              <ul class="flex-container space-evenly">
                <li>
                  <b>Focus: </b>
                  <p>
                    {{ getFocusName(e.focusId) }}
                  </p>
                </li>
                <li>
                  <b>Time: </b>
                  <p>
                    {{ secondsToMinutes(e.time) }}
                  </p>
                </li>
                <li v-if="e.repetitions !== null">
                  <b>Repetitions: </b>
                  <p>
                    {{ e.repetitions }}
                  </p>
                </li>
                <li v-if="e.sets !== null">
                  <b>Sets: </b>
                  <p>
                    {{ e.sets }}
                  </p>
                </li>
                <li v-if="e.weight !== null">
                  <b>Weight: </b>
                  <p>{{ e.weight }} lbs</p>
                </li>
              </ul>
              <div class="flex-container flex-center">
                <v-btn color="success" @click="markExerciseCompleted(index)">{{
                  completedExercises[index]
                    ? "Mark Incomplete"
                    : "Mark Complete"
                }}</v-btn>
              </div>
            </v-expansion-panel-content>
          </div>
        </v-expansion-panel>
      </v-expansion-panels>
      <p v-if="!exercises.length">
        Welcome to your exercises page! From here, you can click on the
        <b>Generate Workout</b> button to generate a new workout,
        <b>Log Workout</b> to log the exercises you have completed, or
        <b>Workout History</b> to view your past workouts.
      </p>
      <v-divider style="padding-top: 0.5rem" />
      <v-card-actions>
        <div class="flex-container">
          <v-btn color="info" :to="{ name: 'workout-history' }"
            >Workout History</v-btn
          >
          <v-btn
            color="info"
            @click="
              completedExercises = completedExercises.some((v) => v)
                ? new Array(exercises.length).fill(false)
                : new Array(exercises.length).fill(true)
            "
            >{{
              completedExercises.some((v) => v)
                ? "Mark all incomplete"
                : "Mark all complete"
            }}</v-btn
          >
          <v-btn
            color="primary"
            @click="logWorkout"
            :disabled="!isAnExerciseCompleted"
            >Log Workout</v-btn
          >
        </div>
      </v-card-actions>
    </v-card-text>
  </v-card>
</template>

<script>
import focusService from "@/services/FocusService";
import exerciseService from "@/services/ExerciseService";
import axios from "axios";
import utilities from "@/utilities";

export default {
  created() {
    this.getFocuses();
    this.getExercises();
  },
  data() {
    return {
      exercises: [],
      focuses: [],
      completedExercises: [],
    };
  },
  computed: {
    focusNames() {
      return this.focuses.map((f) =>
        utilities.capitalizeFirstLetter(f.focusName)
      );
    },
    isAnExerciseCompleted() {
      return this.completedExercises.some((c) => c);
    },
  },
  methods: {
    secondsToMinutes: utilities.secondsToMinutes,
    logWorkout() {
      const time = this.calculateWorkoutLength();
      if (time <= 0) return;
      const workoutHistory = {
        userId: this.$store.state.user.userId,
        time: time,
        date: this.formatLocalISO(new Date()).substr(0, 10),
      };
      console.log({ workoutHistory });
      axios
        .post("/workoutHistory", workoutHistory)
        .then((response) => {
          console.log(response);
          if (response.status >= 200 && response.status < 300) {
            alert("You have succesfully logged your workout");
            this.completedExercises = [];
          }
        })
        .catch((error) => console.log(error));
    },
    formatLocalISO(t) {
      // Ripped from stack overflow :D
      // https://stackoverflow.com/questions/12413243/javascript-date-format-like-iso-but-local
      const z = t.getTimezoneOffset() * 60 * 1000;
      let tLocal = t - z;
      tLocal = new Date(tLocal);
      const iso = tLocal.toISOString();
      return iso.slice(0, 19);
    },
    calculateWorkoutLength() {
      let time = 0;
      for (const exerciseIndex in this.completedExercises) {
        if (this.completedExercises[exerciseIndex]) {
          time += this.exercises[exerciseIndex].time;
        }
      }
      return time;
    },
    markExerciseCompleted(index) {
      this.$set(
        this.completedExercises,
        index,
        !this.completedExercises[index]
      );
    },
    getFocuses() {
      // Get all the focuses
      focusService
        .getFocuses()
        .then((response) => {
          if (response.status === 200) {
            this.focuses = response.data;
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getFocusName(id) {
      let arrayWithFocusObj = this.focuses.filter((f) => {
        return f.focusId === id;
      });
      return utilities.capitalizeFirstLetter(
        (arrayWithFocusObj[0] || { focusName: undefined }).focusName
      );
    },
    getExercises() {
      exerciseService
        .getUserExercises(this.$store.state.user.username)
        .then((response) => {
          if (response.status === 200) {
            this.exercises = response.data;
            //fixParticles();
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
    fixParticles() {
      setTimeout(() => {
        let event = document.createEvent("UIEvents");
        event.initUIEvent("resize", true, false, window, 0);
        window.dispatchEvent(event);
      }, 200); // 200ms delay
    },
  },
};
</script>

<style scoped>
.complete {
  background-color: lightgrey;
}

.flex-container {
  width: 100%;
  justify-content: space-between;
  flex-wrap: wrap;
}

.space-evenly {
  justify-content: space-evenly;
}

.flex-center {
  justify-content: center;
}

.flex-center >>> * {
  margin: 0 0.5rem;
}

li {
  list-style: none;
}

ul {
  padding: 1rem 0;
}

p {
  display: inline;
}
</style>
