<template>
    <div id="app">
        <navbar />
        <div class="content-container">
            <router-view />
        </div>
        <Footer />
    </div>
</template>

<script>
import Navbar from "@/components/Navbar.vue";
import Footer from "@/components/Footer.vue";
import { bus } from "./main";
import { mapGetters } from "vuex";
export default {
    name: "App",
    components: {
        Navbar,
        Footer
    },
    created () {
        this.authenticateUser();
    },
    methods: {
        ...mapGetters(["GET_USER", "GET_USERID"]),
        listenForNotification (data) {
            bus.$emit("changeMessage", `new message from ${data.newMessage}`, "");
            bus.$emit("markUnreadMessage", data.roomId);
        },
        listenForBidNotification (data) {
            bus.$emit("changeMessage", `you have been outbiddet on ${data.auctionName}`, "");
        },
        async authenticateUser () {
            this.$store.watch(() => this.GET_USER().username, () => {
                if (this.GET_USER().username) {
                    this.$sock.emit("joinUserRoom", { userId: this.GET_USERID() });
                    this.$sock.on("chatUserInfo", this.listenForNotification);
                    this.$sock.on("chatUserBidInfo", this.listenForBidNotification);
                } else if (this.GET_USER().username == null) {
                    this.$sock.removeListener("chatUserInfo", this.listenForNotification);
                    this.$sock.removeListener("chatUserBidInfo", this.listenForBidNotification);
                }
            });
            try {
                const response = await this.$http.get("api/auth/isAuthenticated");
                this.$sock.open();
                this.$store.commit("UPDATE_USER", response.data.user);
                this.$store.commit("UPDATE_AVATAR", response.data.avatarImage || "");
                this.$store.commit("UPDATE_USERID", response.data.id);
            } catch (error) {
                this.$store.commit("UPDATE_USER", null);
                this.$store.commit("UPDATE_USERID", null);
            }
        }
    }
};
</script>

<style lang="scss">
body {
    background: $main-light-grey;
    padding: 0;
    margin: 0;
    font-family: "Secular One", sans-serif;
    overflow-y: scroll;
}
#app {
    display: flex;
    flex-direction: column;
}
.content-container {
    position: relative;
    min-height: calc(100vh - 8rem);
}
</style>
