<template>
  <v-form>
    <v-container>
      <v-row
        align="center"
        justify="center"
        class="mt-4"
      >
        <v-col
          sm="8"
        >
          <div
            class="mt-6 pl-4 pr-4"
          >
            <v-row>
              <v-col md="auto">
                <v-card
                  tile
                  :elevation="0"
                >
                  Tenant ID:
                </v-card>
              </v-col>
              <v-col
                md="auto"
                class="ml-auto"
              >
                <v-card
                  class="auto"
                  tile
                  :elevation="0"
                >
                  <v-chip>
                    <span>
                      {{ tenant }}
                    </span>
                    <v-icon
                      v-clipboard="tenant"
                      v-clipboard:success="() => { copySnack = true; }"
                      right
                    >
                      mdi-content-copy
                    </v-icon>
                  </v-chip>
                </v-card>
              </v-col>
            </v-row>
          </div>

          <v-divider />
          <v-divider />

          <div
            class="mt-6 pl-4 pr-4"
          >
            <v-text-field
              v-model="username"
              label="Username"
              type="text"
              :disabled="!editDataStatus"
            />

            <v-text-field
              v-model="email"
              label="E-mail"
              type="text"
              :disabled="!editDataStatus"
            />

            <SettingProfileEditButton
              action="data"
              @statusEdit="enableEdit('editData')"
              @update="updateData('editData')"
            />
          </div>

          <v-divider class="mt-6" />
          <v-divider class="mb-6" />

          <div
            class="mt-6 pl-4 pr-4"
          >
            <v-text-field
              v-model="password"
              :append-icon="show ? 'mdi-eye' : 'mdi-eye-off'"
              :type="show ? 'text' : 'password'"
              label="Password"
              :disabled="!editPasswordStatus"
            />

            <SettingProfileEditButton
              action="password"
              @statusEdit="enableEdit('editPassword')"
              @update="updateData('editPassword')"
            />
          </div>
        </v-col>
      </v-row>
    </v-container>
  </v-form>
</template>

<script>

import SettingProfileEditButton from '@/components/setting/SettingProfileEditButton';

export default {
  name: 'SettingProfile',

  components: {
    SettingProfileEditButton,
  },

  data() {
    return {
      username: '',
      email: '',
      password: '',
      editDataStatus: false,
      editPasswordStatus: false,
      show: false,
    };
  },

  computed: {
    tenant() {
      return this.$store.getters['auth/tenant'];
    },
  },

  created() {
    this.setData();
  },

  methods: {
    setData() {
      this.username = this.$store.getters['auth/currentUser'];
      this.email = this.$store.getters['auth/email'];
    },

    postUser() {
      this.$store.dispatch('users/setUser', 'teste');
    },

    enableEdit(field) {
      if (field === 'editData') {
        this.editDataStatus = !this.editDataStatus;
      } else if (field === 'editPassword') {
        this.editPasswordStatus = !this.editPasswordStatus;
      }
      this.setData();
    },

    updateData(field) {
      const data = {
        username: this.username,
        email: this.email,
        password: this.password,
      };

      this.$store.dispatch('users/put', data);

      this.enableEdit(field);
    },
  },

};
</script>
