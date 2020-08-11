<template>
  <section class="page-section py-5 md:py-10 mb-5">
    <div class="px-5 sm:px-10">
      <div class="list-row-border-b">
        <div class="mr-4">{{ $t("UNIK.OWNER") }}</div>
        <div class="truncate">
          <LinkWallet :address="unik.ownerId" :trunc="false" :simple="true" />
        </div>
      </div>

      <div class="list-row-border-b">
        <div class="mr-4">{{ $t("UNIK.TYPE") }}</div>
        {{ $t(unik.type) }}
      </div>

      <div class="list-row-border-b-no-wrap">
        <div class="mr-4">{{ $t("UNIK.CREATION_DATE") }}</div>
        <div v-if="unik.creation">{{ readableTimestamp(unik.creation) }}</div>
        <div v-else>
          <i>{{ $t("Not yet available") }}</i>
        </div>
      </div>

      <div class="list-row">
        <div class="mr-4">{{ $t("UNIK.LIFECYCLE.LABEL") }}</div>
        <div>{{ $t(lifeCycleStatus) }}</div>
      </div>
    </div>

    <UnikProperties v-if="isDisplayed(getBadges)" :properties="badges" :title="$t('BADGES.LABEL')" />

    <UnikProperties
      v-if="isDisplayed(getUserProperties)"
      :properties="userProperties"
      :title="$t('UNIK.USER_PROPERTIES')"
    />

    <UnikProperties
      v-if="isDisplayed(getSystemProperties)"
      :properties="systemProperties"
      :title="$t('UNIK.SYSTEM_PROPERTIES')"
    />
  </section>
</template>

<script lang="ts">
import { Vue, Prop, Component } from "vue-property-decorator";
import UnikProperties from "@/components/unik/UnikProperties.vue";

export type property = { key: string; value: string };

@Component({
  components: { UnikProperties },
})
export default class UnikDetails extends Vue {
  @Prop({ required: true, default: {} }) public unik: any;

  protected badges = [];
  protected userProperties = [];
  protected systemProperties = [];

  get lifeCycleStatus(): string {
    const status = Object.values(this.unik.properties).find(
      (entry: property) => entry.key === "LifeCycle/Status",
    ) as property;

    switch (status.value) {
      case "1":
        return "UNIK.LIFECYCLE.ISSUED";
      case "2":
        return "UNIK.LIFECYCLE.MINTED";
      case "3":
        return "UNIK.LIFECYCLE.LIVE";
      case "4":
        return "UNIK.LIFECYCLE.ABORTED";
      case "10":
        return "UNIK.LIFECYCLE.EVERLASTING";
      // should not happen
      default:
        return "UNIK.LIFECYCLE.ISSUED";
    }
  }

  get getBadges(): property[] {
    this.badges = Object.values(this.unik.properties)
      .filter((entry: property) => /^Badges\/.*/.test(entry.key))
      .filter((entry: property) => entry.value !== "false");
    return this.badges;
  }

  get getUserProperties(): property[] {
    this.userProperties = Object.values(this.unik.properties)
      .filter((entry: property) => /^usr\/.*/.test(entry.key))
      //remove usr/ prefix from property key
      .map((entry: property) => ({ value: entry.value, key: entry.key.substr(4) }));
    return this.userProperties;
  }

  get getSystemProperties(): property[] {
    this.systemProperties = Object.values(this.unik.properties).filter(
      (entry: property) =>
        !/^Badges\/.*/.test(entry.key) && !/^usr\/.*/.test(entry.key) && entry.key !== "LifeCycle/Status",
    );
    return this.systemProperties;
  }

  private isDisplayed(properties: property[]) {
    return properties && properties.length > 0;
  }
}
</script>

<style scoped>
.list-row-border-b-no-wrap > div:last-child {
  text-align: right;
}
</style>
