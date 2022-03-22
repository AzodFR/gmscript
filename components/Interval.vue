<template>
  <div></div>
</template>

<script>
const time = 15000;
export default {
  name: "Interval",
  async mounted() {
    this.checkRPC();
    this.launchCheck();
    await this.getTemplates();
    this.fetchTokens();
    this.launchFetchStake();
    this.fetchUserInfo();
    this.launchIntervalTokens();
    this.launchIntervalInfo();
    this.fetchItems("tools");
    this.launchIntervalItems();

  },
  methods: {
    launchCheck: async function() {
    setInterval(async () => {
      this.checkRPC()
      },600000)
    },
    checkRPC: async function() {
      let valid = false;

        setTimeout(() => {
          if (valid) {
            valid = false
            console.log("rpc checked")
          }
          else {
            console.log("fuck rpc")
            localStorage.setItem('rpc', 'random');
            if (!localStorage.getItem("autoLogin") ||  localStorage.getItem("autoLogin") == "false"){
            localStorage.setItem("autoLogin", "rpc")
            }
            window.location.href = "/"
          }
        }, 20000);
        try {
        await fetch(
        `${this.$store.state.user.wax.rpc.endpoint}/v1/chain/get_table_rows`,
        {
          credentials: "omit",
          headers: {
            Accept: "*/*",
            "Accept-Language": "fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3",
            "Content-Type": "text/plain;charset=UTF-8",
            "Sec-Fetch-Dest": "empty",
            "Sec-Fetch-Mode": "no-cors",
            "Sec-Fetch-Site": "cross-site",
          },
          referrer: "https://thedefimining.io/",
          body: `{\"json\":true,\"code\":\"galaxyminers\",\"scope\":\"galaxyminers\",\"table\":\"accounts\",\"table_key\":\"\",\"lower_bound\":\"${this.$store.state.user.name}\",\"upper_bound\":\"${this.$store.state.user.name}\",\"limit\":\"100\",\"reverse\":false,\"show_payer\":false}`,
          method: "POST",
          mode: "cors",
        }
      ).then((e) =>  {
        console.log("check", e)
        valid = true}
        )
      .catch((e) => {
        console.log("fuck rpc")
            localStorage.setItem('rpc', 'random');
            if (!localStorage.getItem("autoLogin") ||  localStorage.getItem("autoLogin") == "false"){
            localStorage.setItem("autoLogin", "rpc")
            }
            window.location.href = "/"
      })
        }
        catch (e) {
          onsole.log("fuck rpc")
            localStorage.setItem('rpc', 'random');
            if (!localStorage.getItem("autoLogin") ||  localStorage.getItem("autoLogin") == "false"){
            localStorage.setItem("autoLogin", "rpc")
            }
            window.location.href = "/"
        }

    },
    launchFetchStake: function() {
      this.fetchStake();
      setInterval(() => {
        this.fetchStake();
      }, time)
    },
    fetchStake: async function() {
await fetch("https://chain.wax.io/v1/chain/get_account", {
    "credentials": "omit",
    "headers": {
        "Accept-Language": "fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3",
          "Content-Type": "text/plain;charset=UTF-8",
          "Sec-Fetch-Dest": "empty",
          "Sec-Fetch-Mode": "no-cors",
          "Sec-Fetch-Site": "cross-site",
    },
    "referrer": "https://fw.f12key.xyz/",
    "body": `{\"account_name\":\"${this.$store.state.user.name}\"}`,
    "method": "POST",
    "mode": "cors"
}).then((x) => x.json())
        .then(async (rows) => {
          this.$store.commit("user/setStake", parseFloat(rows.total_resources.cpu_weight.split(" ")[0]).toFixed(2));
          this.$store.commit("user/setCPU", (rows.cpu_limit.used * 100 / rows.cpu_limit.max).toFixed(0));
        })
    },
    launchIntervalItems: function () {
      setInterval(() => {
        this.fetchItems("tools");
      }, time);
    },
    findInTemplate: function (type, id) {
      let template;
      this.$store.state.user.templates[type].forEach((elem) => {
        if (elem.template_id == id) {
          template = elem;
        }
      });
      return template;
    },

    async fetchItems(item) {
      await fetch("https://wax.greymass.com/v1/chain/get_table_rows", {
        credentials: "omit",
        headers: {
          Accept: "*/*",
          "Accept-Language": "fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3",
          "Content-Type": "text/plain;charset=UTF-8",
          "Sec-Fetch-Dest": "empty",
          "Sec-Fetch-Mode": "no-cors",
          "Sec-Fetch-Site": "cross-site",
        },
        referrer: "https://play.galaxyminers.io/",
        body: `{\"json\":true,\"code\":\"galaxyminers\",\"scope\":\"galaxyminers\",\"table\":\"gadgets\",\"table_key\":\"\",\"lower_bound\":\"${this.$store.state.user.name}\",\"upper_bound\":\"${this.$store.state.user.name}\",\"index_position\":2,\"key_type\":\"i64\",\"limit\":\"100\",\"reverse\":false,\"show_payer\":false}`,
        method: "POST",
        mode: "cors",
      })
        .then((x) => x.json())
        .then(async (items) => {

          const newList = {
            "Asteroid": [],
            "Plasma": [],
            "Oxygenk": []
          };
          await items.rows.forEach(async (elem) => {
            const tmp = this.findInTemplate("tools", elem.template_id);
            elem.name = tmp.template_name;
            elem.img = tmp.img;
            elem.durability_usage = tmp.durability_consumed;
            elem.claim_type = tmp.type;
            elem.production = parseInt(tmp.rewards[0].split(' ')[0]);
            elem.power_usage = tmp.energy_consumed;
            elem.charged_time = tmp.charged_time;
            if (!this.$store.state.user.logged_asset.includes(elem.asset_id)) {
              this.$store.commit("user/addAsset", elem.asset_id);
               this.$store.commit("user/addProduction", {
                  type: elem.claim_type.toUpperCase(),
                  value: tmp.type == elem.production,
                });
              this.$store.commit("user/addCost", {
                type: "OXYGEN",
                value: tmp.type == elem.power_usage,
              });
              this.$store.commit("user/addCost", {
                type: "PLASMA",
                value: tmp.type == elem.durability_usage * 0.2,
              });
            }
            if (localStorage.getItem(`${elem.asset_id}`)) {
              if (localStorage.getItem(`${elem.asset_id}`) == "true") {
                this.$store.commit("user/setAutoClaim", {
                  type: elem.claim_type,
                  id: elem.asset_id,
                  value: true,
                });
                this.checker = true;
              } else {
                this.$store.commit("user/setAutoClaim", {
                  type: elem.claim_type,
                  id: elem.asset_id,
                  value: false,
                });
              }
            }
            if (localStorage.getItem(`${elem.asset_id}_r`)) {
              if (localStorage.getItem(`${elem.asset_id}_r`) == "true") {
                this.$store.commit("user/setAutoRepair", {
                  type: elem.claim_type,
                  id: elem.asset_id,
                  value: true,
                });
                this.checker = true;
              } else {
                this.$store.commit("user/setAutoRepair", {
                  type: elem.claim_type,
                  id: elem.asset_id,
                  value: false,
                });
              }
            } else {
              this.$store.commit("user/setAutoRepair", {
                type: elem.claim_type,
                id: elem.asset_id,
                value: false,
              });
            }
            newList[elem.claim_type].push(elem);
          });
          if (newList["Asteroid"].length) {
            this.$store.commit("user/setItem", { value: newList["Asteroid"], type: "Asteroid" });
          }
          if (newList["Plasma"].length) {
            this.$store.commit("user/setItem", { value: newList["Plasma"], type: "Plasma" });
          }
          if (newList["Oxygenk"].length) {
            this.$store.commit("user/setItem", { value: newList["Oxygenk"], type: "Oxygenk" });
          }

        });
    },
    async getTemplates() {
      await fetch(
        `${this.$store.state.user.wax.rpc.endpoint}/v1/chain/get_table_rows`,
        {
          credentials: "omit",
          headers: {
            Accept: "*/*",
            "Accept-Language": "fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3",
            "Content-Type": "text/plain;charset=UTF-8",
            "Sec-Fetch-Dest": "empty",
            "Sec-Fetch-Mode": "no-cors",
            "Sec-Fetch-Site": "cross-site",
          },
          referrer: "https://thedefimining.io/",
          body: `{\"json\":true,\"code\":\"galaxyminers\",\"scope\":\"galaxyminers\",\"table\":\"configtools\",\"table_key\":\"\",\"lower_bound\":\"\",\"upper_bound\":\"\",\"limit\":\"100\",\"reverse\":false,\"show_payer\":false}`,
          method: "POST",
          mode: "cors",
        }
      )
        .then((x) => x.json())
        .then((y) => {
          this.$store.commit("user/setTemplates", {type: "tools", rows: y.rows});
        });
    },
    async launchIntervalInfo() {
      this.fetchUserInfo()
      setInterval(() => {
        this.fetchUserInfo();
      }, time);
    },
    async fetchUserInfo() {
      await fetch(
        `${this.$store.state.user.wax.rpc.endpoint}/v1/chain/get_table_rows`,
        {
          credentials: "omit",
          headers: {
            Accept: "*/*",
            "Accept-Language": "fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3",
            "Content-Type": "text/plain;charset=UTF-8",
            "Sec-Fetch-Dest": "empty",
            "Sec-Fetch-Mode": "no-cors",
            "Sec-Fetch-Site": "cross-site",
          },
          referrer: "https://thedefimining.io/",
          body: `{\"json\":true,\"code\":\"galaxyminers\",\"scope\":\"galaxyminers\",\"table\":\"players\",\"table_key\":\"\",\"lower_bound\":\"${this.$store.state.user.name}\",\"upper_bound\":\"${this.$store.state.user.name}\",\"limit\":\"100\",\"reverse\":false,\"show_payer\":false}`,
          method: "POST",
          mode: "cors",
        }
      )
        .then((x) => x.json())
        .then((user) => {
          for(let i = 0; i < user.rows[0].balances.length; i++) {
             this.$store.commit("user/setRessource", {type: user.rows[0].balances[i].split(' ')[1], value: user.rows[0].balances[i].split(' ')[0]});
          }
          
          this.$store.commit("user/setEnergy", user.rows[0].energy);
          this.$store.commit("user/setMaxEnergy", user.rows[0].max_energy);
        });
    },
    async launchIntervalTokens() {
      this.fetchTokens();
      setInterval(() => {
        this.fetchTokens();
      }, time);
    },
    async fetchTokens() {
      await fetch(
        `${this.$store.state.user.wax.rpc.endpoint}/v1/chain/get_currency_balance`,
        {
          credentials: "omit",
          headers: {
            Accept: "*/*",
            "Accept-Language": "fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3",
            "Content-Type": "text/plain;charset=UTF-8",
            "Sec-Fetch-Dest": "empty",
            "Sec-Fetch-Mode": "no-cors",
            "Sec-Fetch-Site": "cross-site",
          },
          referrer: "https://thedefimining.io/",
          body: `{\"code\":\"galaxytokens\",\"account\":\"${this.$store.state.user.name}\"}`,
          method: "POST",
          mode: "cors",
        }
      )
        .then((x) => x.json())
        .then((y) => {
          const sym = ["GMP", "GMO", "GMA"];
          for (let i = 0; i < 3; i++) {
            if (i >= y.length) {
              this.$store.commit("user/setToken", {
                type: sym[i],
                value: "0.0000",
              });
            } else {
              this.$store.commit("user/setToken", {
                type: y[i].split(" ")[1],
                value: y[i] != undefined ? y[i].split(" ")[0] : "0.0000",
              });
            }
          }
        });

      await fetch(
        `${this.$store.state.user.wax.rpc.endpoint}/v1/chain/get_currency_balance`,
        {
          credentials: "omit",
          headers: {
            Accept: "*/*",
            "Accept-Language": "fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3",
            "Content-Type": "text/plain;charset=UTF-8",
            "Sec-Fetch-Dest": "empty",
            "Sec-Fetch-Mode": "no-cors",
            "Sec-Fetch-Site": "cross-site",
          },
          referrer: "https://thedefimining.io/",
          body: `{\"code\":\"eosio.token\",\"account\":\"${this.$store.state.user.name}\",\"symbol\":\"WAX\"}`,
          method: "POST",
          mode: "cors",
        }
      )
        .then((x) => x.json())
        .then((y) => {
          if (y[0] == undefined) {
            this.$store.commit("user/setToken", {
              type: "WAX",
              value: "0.0000",
            });
          } else {
            this.$store.commit("user/setToken", {
              type: "WAX",
              value: y[0] != undefined ? y[0].split(" ")[0] : "0.0000",
            });
          }
        });
    },
  },
};
</script>
