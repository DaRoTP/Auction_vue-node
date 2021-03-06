    <template>
    <div class="auction-card-live-wrapper">
      <div class="banner" :class="{isWinning}">Winning</div>
      <div class="auction-card-live-container">
          <div class="title-bids-wrapper">
            <div class="main-auction-info">
                <v-image :imageUrl="image" />
                <h3 class="auction-name">{{name}}</h3>
            </div>
            <div class="biding-table">
                <span class="time">
                    <i class="fas fa-clock" />
                    <timer
                        :endDate="new Date(endDate)"
                        v-on:timerStatus="setAuctionStatus"
                    />
                </span>
                <div class="bid-table-wrapper">
                    <bid-card
                        v-for="bid in bids"
                        :key="bid.id"
                        :timeStamp="new Date(bid.timeStamp)"
                        :price="parseFloat(bid.price)"
                        :username="bid.author.username"
                    />
                </div>
            </div>
          </div>
          <div class="input-bid-container">
              <place-bid-input v-if="isAuctionLive" :auctionId="id" />
              <router-link class="go-to-link" :to="'/auction/' + this.id">more</router-link>
          </div>
      </div>
    </div>
</template>

<script>
import BidCard from "@/components/BidCard.vue";
import { mapGetters } from "vuex";
import { PlaceBidInput } from "@/components/Inputs";
import Timer from "@/components/Timer";
import VImage from "@/components/VImage";

export default {
    name: "AuctionCardLive",
    components: {
        BidCard,
        PlaceBidInput,
        VImage,
        Timer
    },
    data () {
        return {
            myBid: "",
            isAuctionLive: false
        };
    },
    props: {
        id: {
            type: String
        },
        name: {
            type: String
        },
        image: {
            type: String
        },
        price: {
            type: Number
        },
        endDate: {
            type: Date
        },
        bids: {
            type: Array
        }
    },
    created () {
        this.bids = this.bids.reverse();
        this.joinBiddingRoom();
        this.isAuctionLive = !this.isEnded;
    },
    beforeDestroy () {
        clearInterval(this.liveCountDownInterval);
        this.leaveBiddingRoom();
    },
    methods: {
        ...mapGetters(["GET_USER"]),
        senfBid (data, roomId) {
            if (roomId === this.id) {
                this.bids.unshift(data);
            }
        },
        joinBiddingRoom () {
            this.$sock.emit("joinBidding", { room: this.id });

            this.$sock.on("sendBid", this.senfBid);
        },
        leaveBiddingRoom () {
            this.$sock.emit("leaveBidding", {
                room: this.id
            });
            this.$sock.removeListener("sendBid", this.senfBid);
        },
        setAuctionStatus (status) {
            this.isAuctionLive = status;
        }
    },
    computed: {
        isWinning () {
            return this.GET_USER().username === this.bids[0].author.username;
        },
        isEnded () {
            const timeNow = new Date();
            const endDateParsed = new Date(this.endDate);
            return timeNow > endDateParsed;
        }
    }
};
</script>

<style lang="scss">
.auction-card-live-wrapper{
    margin-bottom: 3rem;
    margin-top: 2rem;
      .banner {
        background: rgb(161, 243, 120);
        color: rgb(32, 64, 28);
        box-shadow: 0 0 0px 0px rgb(219, 255, 165);
        position: absolute;
        z-index: 0;
        height: 2rem;
        padding: 0.2rem 0.5rem;
        border-radius: 10px 10px 0 0 ;
        transition: 0.3s all ease-in-out;
        &.isWinning {
            transform: translateY(-70%);
            box-shadow: 0 0 3px 2px rgb(219, 255, 165);
        }
    }
}
.auction-card-live-container {
    display: flex;
    flex-direction: column;
    background: white;
    box-shadow: 0 1px 1px 1px rgba(0, 0, 0, 0.361);
    border-radius: 10px;
    overflow: hidden;
    height: 17rem;
    width: 35rem;
    flex-wrap: wrap;
    transition: all 0.3s ease-in-out;
    position: relative;
    z-index: 1;

    @include breaking-point-sm {
        height: 25rem;
        width: 19rem;
    }
    .title-bids-wrapper {
        display: flex;
        flex-direction: row;
        flex-grow: 1;
        @include breaking-point-sm {
            flex-direction: column;
        }

        .main-auction-info {
            background: rgb(248, 248, 248);
            display: flex;
            flex-direction: column;
            overflow: hidden;
            width: 15rem;
            height: 100%;
            @include breaking-point-sm {
                width: 100%;
            }
            img {
                width: 16rem;
                height: 6rem;
                object-fit: cover;
                @include breaking-point-sm {
                    width: 100%;
                }
            }
            .auction-name {
                margin: 0;
                font-size: 1rem;
                margin: 0.1rem 0.3rem;
                flex-grow: 1;
                color: $main-dark-blue;
                max-height: 2rem;
            }
        }

        .biding-table {
            display: flex;
            flex-direction: column;
            padding: 0 0.5rem;
            flex-grow: 1;
            .bid-table-wrapper {
                overflow-y: scroll;
                overflow-x: hidden;
                height: 11.5rem;
                &::-webkit-scrollbar {
                    height: 0.5em;
                    width: 10px;
                }
                &::-webkit-scrollbar-track {
                    background: rgb(216, 217, 241);
                    border-radius: 5px;
                }
                &::-webkit-scrollbar-thumb {
                    background-color: rgb(166, 167, 185);
                    border-radius: 10px;
                }
                .bid-card-container {
                    width: auto;
                    margin: 0.5rem;
                    margin-right: 0.5rem;
                    box-sizing: border-box;
                }
            }
            .time {
                display: flex;
                flex-direction: row;
                align-items: center;
                justify-content: center;
                width: 100%;
                margin-top: 0.5rem;
                color: $main-dark-blue;
                .fas {
                    margin-right: 0.3rem;
                }
            }
        }
    }

    .input-bid-container {
        background: rgb(220, 224, 240);
        height: 3rem;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        align-items: center;
        box-sizing: border-box;

        @include breaking-point-sm {
            grid-column: auto;
        }

        .go-to-link {
            color: $main-dark-blue;
            text-decoration: none;
            margin: 0 0.5rem;
            &:hover {
                text-decoration: underline;
            }
        }
        .place-bid-input {
            margin: 0;
            height: 100%;
            background: transparent;
            border-radius: 0;
            button  {
                height: 100%;
                width: 100%;
            };
            input {
                width: 8rem;
                @include breaking-point-sm {
                    width: 5rem;
                }
            }
        }
    }
}
</style>
