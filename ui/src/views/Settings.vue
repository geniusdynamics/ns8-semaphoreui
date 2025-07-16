<!--
  Copyright (C) 2022 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <cv-grid fullWidth>
    <cv-row>
      <cv-column class="page-title">
        <h2>{{ $t("settings.title") }}</h2>
      </cv-column>
    </cv-row>

    <cv-row v-if="error.getConfiguration">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-configuration')"
          :description="error.getConfiguration"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>

    <cv-row>
      <cv-column>
        <cv-tile light>
          <cv-form @submit.prevent="configureModule">
            <!-- FQDN -->
            <cv-text-input
              :label="$t('settings.semaphoreui_fqdn')"
              placeholder="semaphoreui.example.org"
              v-model.trim="host"
              class="mg-bottom"
              :invalid-message="$t(error.host)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="host"
            />

            <!-- Let’s Encrypt -->
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="isLetsEncryptEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{ $t("settings.disabled") }}</template>
              <template slot="text-right">{{ $t("settings.enabled") }}</template>
            </cv-toggle>

            <!-- HTTP → HTTPS -->
            <cv-toggle
              value="httpToHttps"
              :label="$t('settings.http_to_https')"
              v-model="isHttpToHttpsEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{ $t("settings.disabled") }}</template>
              <template slot="text-right">{{ $t("settings.enabled") }}</template>
            </cv-toggle>

            <!-- Admin user -->
            <cv-text-input
              :label="$t('settings.SEMAPHORE_ADMIN')"
              placeholder="adminuser"
              v-model="SEMAPHORE_ADMIN"
              class="mg-bottom"
              :invalid-message="$t(error.SEMAPHORE_ADMIN)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="SEMAPHORE_ADMIN"
            />
            <cv-text-input
              :label="$t('settings.SEMAPHORE_ADMIN_NAME')"
              placeholder="Admin Name"
              v-model="SEMAPHORE_ADMIN_NAME"
              class="mg-bottom"
              :invalid-message="$t(error.SEMAPHORE_ADMIN_NAME)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="SEMAPHORE_ADMIN_NAME"
            />
            <cv-text-input
              :label="$t('settings.SEMAPHORE_ADMIN_EMAIL')"
              placeholder="admin@email.org"
              v-model="SEMAPHORE_ADMIN_EMAIL"
              class="mg-bottom"
              :invalid-message="$t(error.SEMAPHORE_ADMIN_EMAIL)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="SEMAPHORE_ADMIN_EMAIL"
              type="email"
            />
            <cv-text-input
              :label="$t('settings.SEMAPHORE_ADMIN_PASSWORD')"
              placeholder="V3rY_5e(urE_P@$$w0RD"
              v-model="SEMAPHORE_ADMIN_PASSWORD"
              class="mg-bottom"
              :invalid-message="$t(error.SEMAPHORE_ADMIN_PASSWORD)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="SEMAPHORE_ADMIN_PASSWORD"
              type="password"
            />

            <!-- Advanced accordion -->
            <cv-accordion ref="accordion" class="maxwidth mg-bottom">
              <cv-accordion-item :open="toggleAccordion[0]">
                <template slot="title">{{ $t("settings.advanced") }}</template>
                <template slot="content">
                  <!-- LDAP domain selector moved here -->
                  <NsComboBox
                    v-model.trim="ldap_domain"
                    :autoFilter="true"
                    :autoHighlight="true"
                    :title="$t('settings.ldap_domain')"
                    :label="$t('settings.choose_ldap_domain')"
                    :options="domains_list"
                    :acceptUserInput="false"
                    :showItemType="true"
                    :invalid-message="$t(error.ldap_domain)"
                    :disabled="loading.getConfiguration || loading.configureModule"
                    tooltipAlignment="start"
                    tooltipDirection="top"
                  >
                    <template slot="tooltip">
                      {{ $t("settings.choose_the_ldap_domain_to_use") }}
                    </template>
                  </NsComboBox>
                </template>
              </cv-accordion-item>
            </cv-accordion>

            <!-- Global error bar -->
            <cv-row v-if="error.configureModule">
              <cv-column>
                <NsInlineNotification
                  kind="error"
                  :title="$t('action.configure-module')"
                  :description="error.configureModule"
                  :showCloseButton="false"
                />
              </cv-column>
            </cv-row>

            <!-- Save button -->
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
            >
              {{ $t("settings.save") }}
            </NsButton>
          </cv-form>
        </cv-tile>
      </cv-column>
    </cv-row>
  </cv-grid>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  TaskService,
  IconService,
  PageTitleService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "Settings",
  mixins: [
    TaskService,
    IconService,
    UtilService,
    QueryParamService,
    PageTitleService,
  ],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {
      q: { page: "settings" },
      urlCheckInterval: null,
      host: "",
      isLetsEncryptEnabled: false,
      isHttpToHttpsEnabled: true,
      ldap_domain: "",
      domains_list: [],
      SEMAPHORE_ADMIN_PASSWORD: "",
      SEMAPHORE_ADMIN_NAME: "",
      SEMAPHORE_ADMIN_EMAIL: "",
      SEMAPHORE_ADMIN: "",
      loading: {
        getConfiguration: false,
        configureModule: false,
      },
      error: {
        getConfiguration: "",
        configureModule: "",
        host: "",
        lets_encrypt: "",
        http2https: "",
        ldap_domain: "",
        SEMAPHORE_ADMIN_PASSWORD: "",
        SEMAPHORE_ADMIN_NAME: "",
        SEMAPHORE_ADMIN_EMAIL: "",
        SEMAPHORE_ADMIN: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  created() {
    this.getConfiguration();
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  methods: {
    async getConfiguration() {
      this.loading.getConfiguration = true;
      this.error.getConfiguration = "";
      const taskAction = "get-configuration";
      const eventId = this.getUuid();

      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.getConfigurationAborted
      );
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.getConfigurationCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];
      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.getConfiguration = this.getErrorMessage(err);
        this.loading.getConfiguration = false;
      }
    },
    getConfigurationAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.getConfiguration = this.$t("error.generic_error");
      this.loading.getConfiguration = false;
    },
    getConfigurationCompleted(taskContext, taskResult) {
      const config = taskResult.output;
      this.host = config.host;
      this.isLetsEncryptEnabled = config.lets_encrypt;
      this.isHttpToHttpsEnabled = config.http2https;
      this.domains_list = config.domains_list || [];
      this.ldap_domain = config.ldap_domain || "";
      this.SEMAPHORE_ADMIN = config.semaphore_admin || "";
      this.SEMAPHORE_ADMIN_NAME = config.semaphore_admin_name || "";
      this.SEMAPHORE_ADMIN_EMAIL = config.semaphore_admin_email || "";
      this.loading.getConfiguration = false;
      this.focusElement("host");
    },

    validateConfigureModule() {
      this.clearErrors(this);
      let isValidationOk = true;

      if (!this.host) {
        this.error.host = "common.required";
        if (isValidationOk) this.focusElement("host");
        isValidationOk = false;
      }

      if (!this.SEMAPHORE_ADMIN) {
        this.error.SEMAPHORE_ADMIN = "common.required";
        if (isValidationOk) this.focusElement("SEMAPHORE_ADMIN");
        isValidationOk = false;
      }

      if (!this.SEMAPHORE_ADMIN_PASSWORD) {
        this.error.SEMAPHORE_ADMIN_PASSWORD = "common.required";
        if (isValidationOk) this.focusElement("SEMAPHORE_ADMIN_PASSWORD");
        isValidationOk = false;
      }

      return isValidationOk;
    },
    configureModuleValidationFailed(validationErrors) {
      this.loading.configureModule = false;
      let focusAlreadySet = false;
      for (const ve of validationErrors) {
        this.error[ve.parameter] = this.$t("settings." + ve.error);
        if (!focusAlreadySet) {
          this.focusElement(ve.parameter);
          focusAlreadySet = true;
        }
      }
    },
    async configureModule() {
      if (!this.validateConfigureModule()) return;

      this.loading.configureModule = true;
      const taskAction = "configure-module";
      const eventId = this.getUuid();

      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.configureModuleAborted
      );
      this.core.$root.$once(
        `${taskAction}-validation-failed-${eventId}`,
        this.configureModuleValidationFailed
      );
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.configureModuleCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            host: this.host,
            lets_encrypt: this.isLetsEncryptEnabled,
            http2https: this.isHttpToHttpsEnabled,
            ldap_domain: this.ldap_domain,
            SEMAPHORE_ADMIN: this.SEMAPHORE_ADMIN,
            SEMAPHORE_ADMIN_NAME: this.SEMAPHORE_ADMIN_NAME,
            SEMAPHORE_ADMIN_EMAIL: this.SEMAPHORE_ADMIN_EMAIL,
            SEMAPHORE_ADMIN_PASSWORD: this.SEMAPHORE_ADMIN_PASSWORD,
          },
          extra: {
            title: this.$t("settings.instance_configuration", {
              instance: this.instanceName,
            }),
            description: this.$t("settings.configuring"),
            eventId,
          },
        })
      );
      const err = res[0];
      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.configureModule = this.getErrorMessage(err);
        this.loading.configureModule = false;
      }
    },
    configureModuleAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.configureModule = this.$t("error.generic_error");
      this.loading.configureModule = false;
    },
    configureModuleCompleted() {
      this.loading.configureModule = false;
      this.getConfiguration();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
.mg-bottom {
  margin-bottom: $spacing-06;
}
.maxwidth {
  max-width: 38rem;
}
</style>