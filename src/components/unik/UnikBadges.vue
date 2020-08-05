<template>
  <Loader :data="badges">
    <div v-if="hasDisplayedBadges">
      <div class="list-row">
        <div class="mr-4">{{ $t("BADGES.LABEL") }}</div>
      </div>
      <div class="py-2">
        <TableWrapper v-bind="$attrs" :columns="columns" :rows="badgeDisplay">
          <template slot-scope="data">
            <div v-if="data.column.field === 'key'">{{ $t(data.row.key) }}</div>
            <div v-else-if="data.column.field === 'value'">{{ $t(data.row.value) }}</div>
          </template>
        </TableWrapper>
      </div>
    </div>
  </Loader>
</template>

<script lang="ts">
import { Vue, Prop, Component } from "vue-property-decorator";
import { property } from "./Details.vue";

@Component
export default class UnikBadges extends Vue {
  @Prop({ required: true, default: [] }) public badges: property[];

  private badgeDisplay = [];
  get columns() {
    const columns = [
      {
        label: this.$t("BADGES.LABEL"),
        field: "key",
        thClass: "text-right w-64",
        tdClass: "text-right w-64",
      },
      {
        label: this.$t("UNIK.VALUE"),
        field: "value",
        thClass: "text-left",
        tdClass: "text-left",
      },
    ];
    return columns;
  }

  private getBadgeValue(badge): string | undefined {
    switch (badge.key) {
      case "Badges/Security/SecondPassphrase":
        return "BADGES.SECOND_PASSPHRASE_VALUE";
      case "Badges/NP/Delegate":
        return "BADGES.NETWORKPLAYER_VALUE";
      case "Badges/Pioneer":
        return badge.value === "1" ? "BADGES.PIONEER_INNOVATOR" : "BADGES.PIONEER_EARLY";
    }
  }

  private getBadgeKey(badge): string | undefined {
    switch (badge.key) {
      case "Badges/Security/SecondPassphrase":
        return "BADGES.SECOND_PASSPHRASE_KEY";
      case "Badges/NP/Delegate":
        return "BADGES.NETWORKPLAYER_KEY";
      case "Badges/Pioneer":
        return "BADGES.PIONEER_KEY";
    }
  }

  get hasDisplayedBadges() {
    this.badgeDisplay = this.badges
      .filter((badge) => badge.value !== "false")
      .map((badge) => ({
        key: this.getBadgeKey(badge) || badge.key,
        value: this.getBadgeValue(badge) || badge.value,
      }));
    return this.badgeDisplay.length > 0;
  }
}
</script>
