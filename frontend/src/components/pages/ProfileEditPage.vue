<template>
  <teleport to="#nav-title">{{ t("pages.profile.title") }}</teleport>
  <v-main class="mx-8 d-flex flex-column">
    <!-- welcome message on registration -->
    <div class="mt-8" v-if="!authStore.isRegistered">
      <h1 class="font-weight-regular">
        {{ t("pages.profile.welcome") }}
        <span style="color: #b39ddb">matchy</span>.
      </h1>
      <p class="text-grey">{{ t("pages.profile.welcome-subtitle") }}</p>
    </div>

    <v-form class="mt-8" v-model="valid" @submit.prevent="onSubmit.handler">
      <!-- text input fields -->
      <div>
        <v-text-field
          v-model="formData.email"
          label="Email"
          :disabled="true"
          prepend-icon="mdi-email"
        />
        <v-text-field
          v-model="formData.fullName"
          :label="t('pages.profile.full-name-label')"
          :placeholder="t('pages.profile.full-name-placeholder')"
          :disabled="loadingProfile.loading"
          :rules="[(v) => !!v || t('pages.profile.full-name-required')]"
          prepend-icon="mdi-account"
        />
        <v-textarea
          class="mt-3"
          v-model="formData.description"
          :label="t('pages.profile.description-label')"
          :placeholder="t('pages.profile.description-placeholder')"
          :disabled="loadingProfile.loading"
          variant="outlined"
        />
      </div>

      <div class="d-flex flex-column align-center mt-8">
        <!-- update button -->
        <v-btn
          class="mb-4"
          size="x-large"
          color="green"
          variant="tonal"
          append-icon="mdi-account-arrow-left"
          rounded="pill"
          type="submit"
          minWidth="20rem"
          :disabled="onSubmit.loading || !valid"
          :loading="onSubmit.loading"
        >
          {{
            authStore.isRegistered
              ? t("pages.profile.update-button-text")
              : t("pages.profile.register-button-text")
          }}
        </v-btn>

        <!-- sign out button -->
        <v-btn
          v-if="!authStore.isRegistered"
          class="mb-4"
          size="x-large"
          color="secondary"
          variant="tonal"
          append-icon="mdi-logout"
          rounded="pill"
          minWidth="20rem"
          @click="logout"
          >{{ t("pages.profile.sign-out-button-text") }}
        </v-btn>

        <!-- sign delete -->
        <v-btn
          v-if="authStore.isRegistered"
          id="delete-button"
          size="x-large"
          color="error"
          variant="tonal"
          append-icon="mdi-account-off"
          rounded="pill"
          minWidth="20rem"
          @click="deleteDialog = true"
          >{{ t("pages.profile.delete-button-text") }}
          <v-dialog class="dialog" v-model="deleteDialog">
            <v-card>
              <v-card-title class="text-h6">Delete account</v-card-title>
              <v-card-text>
                {{ t("pages.profile.delete-dialog-text") }}
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  color="green darken-1"
                  text
                  @click="deleteDialog = false"
                >
                  {{ t("pages.profile.delete-dialog-keep") }}
                </v-btn>
                <v-btn
                  color="red darken-1"
                  text
                  @click="
                    deleteDialog = false;
                    deleteProfile();
                  "
                >
                  {{ t("pages.profile.delete-dialog-delete") }}
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-btn>
      </div>
    </v-form>
  </v-main>

  <!-- dialog for account deletion -->
</template>

<script setup lang="ts">
import { useAuthStore } from "@/stores/auth";
import { useProfileService, type Profile } from "@/services/profileService";
import { useI18n } from "vue-i18n";

const { t } = useI18n();

const authStore = useAuthStore();
const router = useRouter();
const profileService = useProfileService();

const formData = ref<Profile>({});
const valid = ref(false);
const deleteDialog = ref(false);

// immediately call loadingProfile, update loading state
const loadingProfile = asyncLoading(() =>
  profileService
    .readProfile()
    .then((profile) => {
      formData.value.email = profile.email;
      formData.value.fullName = profile.fullName;
      formData.value.description = profile.description;
    })
    .catch((e) => {
      errorToast(e);
    })
);
loadingProfile.handler();

const onSubmit = asyncLoading(async () => {
  try {
    await profileService.updateProfile(formData.value);
    successToast(t("pages.profile.update-success"));
  } catch (error: any) {
    errorToast(error);
  }
});

function logout() {
  authStore.logout();
  router.push("/login");
}

async function deleteProfile() {
  await authStore.deleteAccount();
  successToast(t("pages.profile.delete-success"));
  router.push("/login");
}
</script>
