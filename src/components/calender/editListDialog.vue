<template>
  <v-dialog v-model="showDialog" :overlay-opacity="1" scrollable>
    <v-card>
      <v-card-text class="pa-5">
        <v-container>
          <v-form ref="form" v-model="formValidation" lazy-validation>
            <v-row>
              <v-col cols="12">
                <v-text-field
                  label="List Name*"
                  v-model="eventObject.listName"
                  :rules="requiredRule"
                  required
                ></v-text-field>
              </v-col>
              <v-col cols="12" sm="4" md="4">
                <v-menu
                  v-model="scheduleFromDatePick"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="eventObject.scheduleFrom"
                      label="Schedule From*"
                      :rules="requiredRule"
                      required
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="eventObject.scheduleFrom"
                    :min="new Date().toISOString()"
                    :max="eventObject.scheduleTo"
                    @input="
                      scheduleFromDatePick = false;
                      scheduleFromTimePick = true;
                    "
                  ></v-date-picker>
                </v-menu>
              </v-col>
              <v-col cols="12" sm="2" md="2">
                <v-menu
                  ref="startTime"
                  v-model="scheduleFromTimePick"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  :return-value.sync="eventObject.scheduleFromTime"
                  transition="scale-transition"
                  offset-y
                  max-width="290px"
                  min-width="290px"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="eventObject.scheduleFromTime"
                      label="Time*"
                      :rules="requiredRule"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-time-picker
                    v-if="scheduleFromTimePick"
                    v-model="eventObject.scheduleFromTime"
                    full-width
                    @click:minute="
                      $refs.startTime.save(eventObject.scheduleFromTime)
                    "
                  ></v-time-picker>
                </v-menu>
              </v-col>
              <v-col cols="12" sm="4" md="4">
                <v-menu
                  v-model="scheduleToDatePick"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  transition="scale-transition"
                  offset-y
                  min-width="auto"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="eventObject.scheduleTo"
                      label="Schedule To*"
                      :rules="requiredRule"
                      required
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-date-picker
                    v-model="eventObject.scheduleTo"
                    :min="
                      eventObject.scheduleFrom === ''
                        ? new Date().toISOString()
                        : eventObject.scheduleFrom
                    "
                    @input="
                      scheduleToDatePick = false;
                      scheduleToTimePick = true;
                    "
                  ></v-date-picker>
                </v-menu>
              </v-col>
              <v-col cols="12" sm="2" md="2">
                <v-menu
                  ref="endTime"
                  v-model="eventObject.scheduleToTimePick"
                  :close-on-content-click="false"
                  :nudge-right="40"
                  :return-value.sync="eventObject.scheduleToTime"
                  transition="scale-transition"
                  offset-y
                  max-width="290px"
                  min-width="290px"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                      v-model="eventObject.scheduleToTime"
                      label="Time*"
                      :rules="requiredRule"
                      readonly
                      v-bind="attrs"
                      v-on="on"
                    ></v-text-field>
                  </template>
                  <v-time-picker
                    v-if="scheduleToTimePick"
                    v-model="eventObject.scheduleToTime"
                    full-width
                    @click:minute="$refs.endTime.save(scheduleToTime)"
                  ></v-time-picker>
                </v-menu>
              </v-col>
              <v-col cols="12">
                <v-text-field
                  v-model="task"
                  label="Enter task*"
                  append-icon="mdi-plus-circle"
                  @keydown.enter="addTask()"
                  @click:append="addTask()"
                ></v-text-field>
              </v-col> </v-row
          ></v-form>

          <v-card v-if="eventObject.tasks.length > 0" class="elevation-0 mx-n4">
            <v-list-item v-for="(task, i) in eventObject.tasks" :key="i">
              <v-list-item-title>
                <v-checkbox
                  class="ml-2"
                  color="accent"
                  v-model="task.done"
                  :label="task.text"
                ></v-checkbox
              ></v-list-item-title>
              <v-icon
                class="ml-1"
                size="20"
                color="error"
                @click="removeTask(task)"
              >
                mdi-close
              </v-icon>
            </v-list-item>
          </v-card>
          <v-card-title
            v-else
            class="elevation-0 subtitle-1 pa-0 justify-center error--text"
            >Please add a task to the list</v-card-title
          >
        </v-container>
      </v-card-text>
      <v-card-actions class="accent">
        <v-spacer></v-spacer>
        <v-btn
          :class="$vuetify.theme.dark ? 'white--text' : 'black--text'"
          text
          @click="showDialog = false"
        >
          Close
        </v-btn>
        <v-btn
          :class="$vuetify.theme.dark ? 'white--text' : 'black--text'"
          text
          @click="saveList"
        >
          Save
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>
<script>
export default {
  name: "edit-list-component",
  data: () => ({
    scheduleFromDatePick: false,
    scheduleFromTimePick: false,
    scheduleToDatePick: false,
    scheduleToTimePick: false,
    formValidation: true,
    task: null,
    requiredRule: [(v) => !!v || "This is a required field."],
  }),
  props: { show: Boolean, eventData: Object },

  computed: {
    showDialog: {
      get() {
        return this.show;
      },
      set() {
        this.$emit("closeDialog");
      },
    },
    eventObject() {
      return this.eventData;
    },
  },
  methods: {
    addTask() {
      if (this.task) {
        this.eventObject.tasks.push({
          done: false,
          text: this.task,
        });
      }
      this.task = null;
    },
    removeTask(task) {
      this.eventObject.tasks.splice(this.eventObject.tasks.indexOf(task), 1);
    },
    validate() {
      this.$refs.form.validate();
    },
    saveList() {
      if (this.$refs.form.validate() && this.eventObject.tasks.length !== 0) {
        if (localStorage.getItem("calenderEvents") !== null) {
          var calenderEvents = JSON.parse(
            localStorage.getItem("calenderEvents")
          );
          var foundIndex = calenderEvents.findIndex(
            (list) => list.id === this.eventObject.id
          );
          calenderEvents[foundIndex] = {
            id: this.eventObject.id,
            name: this.eventObject.listName,
            start:
              this.eventObject.scheduleFrom +
              " " +
              this.eventObject.scheduleFromTime,
            end:
              this.eventObject.scheduleTo +
              " " +
              this.eventObject.scheduleToTime,
            tasks: this.eventObject.tasks,
            color: calenderEvents[foundIndex].color,
          };
          localStorage.setItem(
            "calenderEvents",
            JSON.stringify(calenderEvents)
          );
          this.$emit("updateEvents");
          this.closeDialog();
        }
      }
    },
    closeDialog() {
      this.showDialog = false;
      this.$refs.form.resetValidation();
    },
  },
};
</script>
