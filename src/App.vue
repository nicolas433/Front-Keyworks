<template>
  <v-app>
    <div class="fluid container">
      <div class="card-groups">
        <v-col
          cols="2"
          class="collumn"
          v-for="(cardGroup, index) in draggableList"
          :key="index"
        >
          <div class="collumn-header">
            <v-row justify="space-between" align="center">
              <h4 class="card-group-name">{{ cardGroupList[index].name }}</h4>

              <div class="card-group-infos">
                <v-icon color="white" small>mdi-clock</v-icon>
                <h5>1h</h5>
                <h4 class="card-length">{{ cardGroupList[index].cards.length }}</h4>
              </div>
            </v-row>
          </div>

          <v-container padding="0px">
            <draggable
              :list="draggableList[index]"
              group="cards"
              :move="onMove"
              :id="index"
            >
              <v-card
                v-for="card in draggableList[index]"
                :key="card.id"
                class="card"
              >
                <v-container>
                  <v-row style="max-height: 40px">
                    <v-col class="side-padding-zero">
                      <h4 class="team-type">
                        {{ card.team.type }}
                      </h4>
                    </v-col>
                    <v-col align="right">
                      <h5 class="mini-text">Código:</h5>
                      <h4 class="card-id">#{{ card.id }}</h4>
                    </v-col>
                  </v-row>
                  <v-row class="card-title" style="max-height: 20px;">
                    {{ card.title }}
                  </v-row>
                  <v-row style="max-height: 80px; margin-top: 5px;">
                    <v-col class="side-padding-zero">
                      <h5 class="mini-text">Projeto:</h5>
                      <h4 class="card-name">{{ card.project.name }}</h4>
                    </v-col>
                    <v-col align="right">
                      <h5 class="mini-text">Prevista:</h5>
                      <h4 class="card-dead-line">
                        <v-icon color="#000000B3">mdi-calendar</v-icon>
                        {{ card.dead_line }}
                      </h4>
                    </v-col>
                  </v-row>
                  <v-row style="display: flex; flex-direction: column">
                    <h5 class="mini-text">Descrição:</h5>
                    <h4 class="card-description">
                      {{ formatDescription(card.description) }}
                    </h4>
                  </v-row>
                </v-container>
                <fieldset class="Produtos">
                  <legend>Acompanhamento</legend>
                </fieldset>
                <v-container>
                  <v-row align="center" justify="space-between" style="max-height:40px">
                    <v-col cols="1" style="margin-right: 5px">
                      <v-icon color="#000000B3" width="16px">mdi-clock</v-icon>
                    </v-col>
                    <v-col cols="5" align="left" marginLeft="20px">
                      <h5 class="mini-text">Previsto</h5>
                      <h4 class="card-prevision">{{ card.prevision }}</h4>
                    </v-col>
                    <v-col cols="4" align="center" style="margin-right: 20px">
                      <h4 :class="`status-${statusClass(card.status)}`">
                        {{ card.status }}
                      </h4>
                    </v-col>
                  </v-row>
                  <v-row align="center">
                    <v-col cols="1" style="margin-right: 5px">
                      <v-icon color="#000000B3" width="16px">mdi-clock</v-icon>
                    </v-col>
                    <v-col>
                      <h5 class="mini-text">Saldo</h5>
                      <h4
                        :class="`balance-${balanceClass(
                          card.balance,
                          card.status
                        )}`"
                      >
                        {{ card.balance }}
                      </h4>
                    </v-col>
                    <v-col align="end">
                      <h5 class="mini-text">Equipe</h5>
                      <h4 class="card-team">
                        <div
                          v-for="user in card.team.users"
                          :key="user.id"
                          class="user"
                        >
                          <i>
                            {{ user.name.substr(0, 2) }}
                          </i>
                        </div>
                      </h4>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card>
            </draggable>

            <v-btn
              color="primary"
              class="ma-2"
              dark
              @click="
                () => {
                  newCard_CardGroupId = cardGroup.id;
                  openCardForm = true;
                }
              "
            >
              Novo cartão
            </v-btn>
          </v-container>
        </v-col>

        <v-col cols="2" class="new-card-group">
          <div>
            <v-btn
              color="primary"
              class="ma-2"
              dark
              @click="openCardGroupForm = true"
            >
              Novo grupo
            </v-btn>
          </div>
        </v-col>
      </div>

      <v-dialog v-model="openCardGroupForm" max-width="500px">
        <v-card>
          <v-card-title> Novo grupo de cartões </v-card-title>

          <form @submit.prevent="createCardGroup">
            <v-card-text>
              <v-text-field
                v-model="newCardGroup_Name"
                label="Nome"
                required
              ></v-text-field>
            </v-card-text>
            <v-card-actions
              style="display: flex; justify-content: space-between"
            >
              <v-btn depressed color="error" @click="openCardGroupForm = false">
                Fechar
              </v-btn>
              <v-btn depressed color="success" type="submit"> Salvar </v-btn>
            </v-card-actions>
          </form>
        </v-card>
      </v-dialog>

      <v-dialog v-model="openCardForm" max-width="500px">
        <v-card>
          <v-card-title> Novo cartão </v-card-title>
          <form @submit.prevent="createCard">
            <v-card-text>
              <v-text-field v-model="newCard_Title" label="Nome" required />

              <v-text-field
                v-model="newCard_DeadLine"
                label="Data de Entrega"
                type="date"
                required
              />

              <v-textarea
                v-model="newCard_Description"
                name="input-7-1"
                label="Descrição"
                required
              />

              <v-text-field
                v-model="newCard_Balance"
                label="Saldo"
                type="time"
                min="0"
                required
              />

              <v-select
                v-model="newCard_Status"
                :items="statusList"
                label="Status"
                data-vv-name="select"
                required
              />

              <v-text-field
                v-model="newCard_Prevision"
                label="Previsão"
                type="time"
                min="0"
                required
              />

              <v-select
                v-model="newCard_ProjectId"
                :items="projectList"
                label="Projeto"
                data-vv-name="select"
                item-text="name"
                item-value="id"
                required
              />

              <v-select
                v-model="newCard_TeamId"
                :items="teamsList"
                label="Time"
                data-vv-name="select"
                item-text="name"
                item-value="id"
                required
              />
            </v-card-text>

            <v-card-actions
              style="display: flex; justify-content: space-between"
            >
              <v-btn
                depressed
                color="error"
                @click="
                  () => {
                    openCardForm = false;
                    newCard_GroupId = false;
                  }
                "
              >
                Fechar
              </v-btn>
              <v-btn type="submit" depressed color="success"> Salvar </v-btn>
            </v-card-actions>
          </form>
        </v-card>
      </v-dialog>
    </div>
  </v-app>
</template>

<script>
import draggable from "vuedraggable";
import axios from "axios";

export default {
  name: "app",
  components: {
    draggable,
  },
  data() {
    return {
      cardGroupList: [],
      teamsList: [],
      projectList: [],
      statusList: ["EM DIA", "EM ATRASO", "ATENÇÃO"],

      editable: true,
      isDragging: false,
      delayedDragging: false,
      draggableList: [],

      openCardGroupForm: false,
      openCardForm: false,

      newCardGroup_Name: "",

      newCard_Title: "",
      newCard_DeadLine: "",
      newCard_Description: "",
      newCard_Balance: "",
      newCard_Status: "",
      newCard_Prevision: false,
      newCard_CardGroupId: false,
      newCard_ProjectId: false,
      newCard_TeamId: false,
    };
  },
  methods: {
    onMove({ draggedContext, to }) {
      this.updateCard_CardGroup(
        draggedContext.element.id,
        this.cardGroupList[to.id].id
      );
    },
    updateCard_CardGroup(cardId, to) {
      axios
        .post("http://localhost/api/cards/card-group", {
          id: cardId,
          card_group_id: to,
        })
        .then(() => {});
    },
    updateCard(card) {
      axios.put("http://localhost/api/cards", { card }).then((response) => {
        this.projectList = response.data.list;
      });
    },
    getProjects() {
      axios.get("http://localhost/api/projects").then((response) => {
        this.projectList = response.data.list;
      });
    },
    getTeams() {
      axios.get("http://localhost/api/teams").then((response) => {
        this.teamsList = response.data.list;
      });
    },
    getCardsGroups() {
      axios.get("http://localhost/api/card-groups").then((response) => {
        const { list } = response.data;

        this.cardGroupList = list;
        this.draggableList = list.map((cardGroup) => {
          return [...cardGroup.cards];
        });
      });
    },
    createCard() {
      try {
        axios
          .post("http://localhost/api/cards", {
            title: this.newCard_Title,
            dead_line: this.newCard_DeadLine,
            description: this.newCard_Description,
            balance: this.newCard_Balance,
            status: this.newCard_Status,
            prevision: this.newCard_Prevision,
            card_group_id: this.newCard_CardGroupId,
            project_id: this.newCard_ProjectId,
            team_id: this.newCard_TeamId,
          })
          .then((response) => {
            this.openCardForm = false;
            if (response.status === 201) {
              this.getCardsGroups();
            }
          })
          .finally(() => {
            this.newCard_Title = "";
            this.newCard_DeadLine = "";
            this.newCard_Description = "";
            this.newCard_Balance = "";
            this.newCard_Status = "";
            this.newCard_Prevision = false;
            this.newCard_CardGroupId = false;
            this.newCard_ProjectId = false;
            this.newCard_TeamId = false;
          });
      } catch (error) {
        console.log(error);
      }
    },
    createCardGroup() {
      try {
        axios
          .post("http://localhost/api/card-groups", {
            name: this.newCardGroup_Name,
          })
          .then((response) => {
            this.openCardGroupForm = false;
            this.getCards;
            if (response.status === 201) {
              this.cardGroupList = [
                ...this.cardGroupList,
                { ...response.data.data, cards: [] },
              ];
            }
          })
          .finally(() => {
            this.newCardGroup_Name = "";
          });
      } catch (error) {
        console.log(error);
      }
    },
    formatDescription(description) {
      return `${description.substr(0, 40)}...`;
    },
    statusClass(status) {
      switch (status) {
        case "EM DIA":
          return "alright";
        case "ATENÇÃO":
          return "warning";
        case "EM ATRASO":
          return "danger";
      }
    },
    balanceClass(balance, status) {
      const state = balance.substr(0, 1);

      if (state === "+") {
        if (status === "ATENÇÃO") {
          return "warning";
        }
        return "alright";
      }

      return "danger";
    },
  },
  created() {
    this.getCardsGroups();
    this.getTeams();
    this.getProjects();
  },
};
</script>

<style>
#app {
  font-family: "Montserrat", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50 !important;
}

.collumn {
  min-width: 320px;
  margin-right: 20px;
  padding: 0px;
  border-radius: 10px 0px 10px 0px;
  background-color: #eaedea;
}

.collumn-header {
  background-color: #8cc587;
  color: white;
  font-size: 16px;
  width: 100%;
  height: 45px;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  border-radius: 10px 0px 10px 0px;
  padding-left: 15px;
}

.card-groups {
  display: flex;
  flex-direction: row;
  padding-top: 50px;
  overflow: auto;
}

.card-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 0px;
  padding-bottom: 6px;
}

.card {
  cursor: move;
  min-height: 250px;
  margin-bottom: 20px;
  padding-left: 10px;
  padding-right: 10px;
  overflow: hidden;
}

.new-card-group {
  min-width: 320px;
  height: 400px;
  margin-right: 20px;
  padding: 0px;
  border-radius: 10px 0px 10px 0px;
  background-color: #eaedea;
  display: flex;
  justify-content: center;
  align-items: center;
}

.custom-form {
  display: flex;
  justify-content: center;
  flex-direction: column;
  padding: 10px;
}

.team-type {
  font-size: 11px;
  border: 1px solid #00000080;
  border-radius: 2px;
  width: fit-content;
  padding-right: 3px;
  padding-left: 3px;
  color: #2c3e50 !important;
}

.mini-text {
  color: #00000080;
  font-size: 10px;
}

.card-id {
  color: #000000e6;
  font-size: 12px;
  font-weight: normal;
}

.card-title {
  color: #000000e6;
  font-size: 14px;
  font-weight: bold;
}

.card-name {
  color: #000000e6;
  font-size: 14px;
  font-weight: normal;
}

.card-dead-line {
  color: #000000e6;
  font-size: 14px;
  font-weight: normal;
}

.card-description {
  color: #000000e6;
  font-size: 12px;
  font-weight: normal;
  text-align: justify;
  max-height: 40px;
  overflow: hidden;
}

.card-prevision {
  color: #000000e6;
  font-size: 11px;
  font-weight: normal;
}

.align-right {
  text-align: right;
}

.align-left {
  text-align: left;
}

.side-padding-zero {
  padding-right: 0px !important;
  padding-left: 0px !important;
}

.status-alright {
  background-color: #107154;
  width: fit-content;
  height: 24px;
  width: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  font-weight: bolder;
  font-size: 11px;
  color: white;
  padding: 0px;
}

.status-warning {
  background-color: #f7e702;
  height: 24px;
  width: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  font-weight: bolder;
  font-size: 11px;
  color: #000000b3;
  padding: 0px;
}

.status-danger {
  background-color: #a31e20;
  height: 24px;
  width: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  font-weight: bolder;
  font-size: 11px;
  color: white;
  padding: 0px;
}

.balance-alright {
  border: 2px solid #107154;
  width: fit-content;
  height: fit-content;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  font-weight: bolder;
  font-size: 11px;
  color: #107154;
  padding-right: 3px;
  padding-left: 3px;
  padding-top: 2px;
}

.balance-warning {
  background-color: #f7e702;
  color: #000000b3;
  width: fit-content;
  height: fit-content;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  font-weight: bolder;
  font-size: 11px;
  padding-right: 3px;
  padding-left: 3px;
  padding-top: 2px;
}

.balance-danger {
  border: 2px solid #a31e20;
  width: fit-content;
  height: fit-content;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  font-weight: bolder;
  font-size: 11px;
  color: #a31e20;
  padding-right: 3px;
  padding-left: 3px;
  padding-top: 2px;
}

.card-team {
  display: flex;
  align-items: flex-end;
  justify-content: flex-end;
  text-align: end;
}

.user {
  background-color: #00000080;
  color: white;
  margin: 2px;
  font-size: 10px;
  height: 20px;
  width: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}

.card-group-name {
  padding-left: 10px;
}

.card-group-infos {
  width: 100px;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  margin-left: -20px;
}

.card-length {
  background: white;
  color: #8CC587;
  margin-left: 20px;
  height: 20px;
  width: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  font-size: 14px;
}

fieldset legend {
  font-size: 10px;
  color: #0000004d !important;
  margin-left: 5px;
  margin-right: 5px;
}

fieldset {
  border-top: 1px solid #00000029;
  border-bottom: none;
  border-left: none;
  border-right: none;
  display: block;
  text-align: left;
  padding-left: 10px;
  margin-top: 10px;
  width: 300px;
  margin-left: -10px;
}
</style>
