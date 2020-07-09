<template>
  <v-form>
    <v-btn
      v-if="!editStatus"
      class="mr-2"
      outlined
      @click="enableEdit()"
    >
      Change {{ action }}
    </v-btn>

    <div
      v-if="editStatus"
    >
      <v-btn
        class="mr-2"
        outlined
        @click="enableEdit()"
      >
        Cancel
      </v-btn>

      <v-btn
        class="mr-2"
        outlined
        @click="save()"
      >
        Save
      </v-btn>
    </div>
  </v-form>
</template>

<script>

export default {
  name: 'SettingProfileEditButton',

  props: {
    action: {
      type: String,
      default: 'data',
      validator: (value) => ['data', 'password'].includes(value),
    },
  },

  data() {
    return {
      editStatus: false,
    };
  },

  methods: {
    enableEdit() {
      this.editStatus = !this.editStatus;
      this.$emit('statusEdit');
    },

    save() {
      this.editStatus = !this.editStatus;
      this.$emit('update');
    },
  },

};
</script>
