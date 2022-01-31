<template>
  <div class="user">
    <div class="description">
      <div class="name">
        {{ user.username }} {{ user.self ? " (yourself)" : "" }}
      </div>
      <div class="status">
        <status-icon :connected="user.connected" />{{ status }}
      </div>
    </div>
  </div>
</template>

<script>
import StatusIcon from "./StatusIcon";
export default {
  name: "User",
  components: { StatusIcon },
  props: {
    user: Object,
    selected: Boolean,
  },
  methods: {
  },
  computed: {
    status() {
      return this.user.connected ? "online" : "offline";
    },
  },
};
</script>

<style scoped>
.selected {
  background-color: #1164a3;
}

.user {
  padding: 10px;
}

.description {
  display: inline-block;
}

.status {
  color: #92959e;
}
</style>
