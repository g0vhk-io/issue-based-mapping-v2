<template>
<div>
<v-container grid-list-md>
    <v-layout row>
      <v-flex xs9>
        <v-card flat class="mt-2">
          <v-card-text>
            <div class="headline"># {{board.name}} 
            </div>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>
    <v-layout row wrap>
      <v-flex md6>
        <v-tabs v-model="tab" left>
          <v-tabs-slider color="indigo"></v-tabs-slider>
          <v-tab href="#tab-2" style="font-size: 1.2rem" :to="{name:'board', params:{id:board.id}}">
            議題分析表
          </v-tab>
          <v-tab href="#tab-1" style="font-size: 1.2rem">
            心智圖
          </v-tab>
        </v-tabs>
      </v-flex>
    </v-layout>
</v-container>
  <div id="mindmap">
   <!--  <v-card flat style="position:absolute;z-index:200" class="mt-3 ml-3">
      <v-card-title primary-title>
        <div class="headline"># {{board.name}} </div>
      </v-card-title>
    </v-card> -->
    <v-card style="position:absolute;z-index:200;margin-top:8em" class="ml-4">
      <v-list>
        <v-list-tile>
          <v-icon v-if="noteicon == false" color="black" @click="opencard = true; noteicon = true">note</v-icon>
          <v-icon v-if="noteicon == true" color="blue" @click="opencard = true; noteicon = false">note</v-icon>
        </v-list-tile>
        <v-list-tile>
          <v-icon v-if="linkicon == false" color="black" @click="pressshift = true; linkicon = true; unlinkicon = false">fa-link</v-icon>
          <v-icon v-if="linkicon == true" color="blue" @click="pressshift = false; linkicon = false">fa-link</v-icon>
        </v-list-tile>
        <v-list-tile>
          <v-icon v-if="unlinkicon == false" color="black" @click="pressshift = true; unlinkicon = true; linkicon = false">fa-unlink</v-icon>
          <v-icon v-if="unlinkicon == true" color="blue" @click="pressshift = false; unlinkicon = false">fa-unlink</v-icon>
        </v-list-tile>
      </v-list>
    </v-card>
    <!-- <v-btn absolute dark fab bottom right color="pink" style="bottom:1em" @click="dialog = true">
      <v-icon medium>add</v-icon>
    </v-btn> -->
    <v-btn absolute dark fab bottom left color="blue" style="bottom:1em" :to="{name:'board', params:{id:board.id}}">
      <v-icon medium>arrow_back</v-icon>
    </v-btn>
    <!-- <v-textarea id="textbox" v-model="textbox.text" solo no-resize outline color="black" :width="textbox.width" :height="textbox.height" v-show="textbox.show"></v-textarea> -->
    <v-stage id="stage" ref="stage" :config="getstageconfig()">
      <v-layer ref="layer">
        <v-group v-for="list in lists" :key="list.id">
          <!-- <v-group v-for="(card, index) in list.cards" :ref="card.id" :key="card.id" @click="linkcard(card)" @dragmove="adjustPoint(card.id)" @dragend="changeposition(card,list)" @mouseenter="showexplain(card,true)" @mouseleave="showexplain(card,false)" :config="getgroupconfig(card)"> -->
          <v-group v-for="(card, index) in list.cards" :ref="card.id" :key="card.id" @click="linkcard(card)" @dragmove="adjustPoint(card.id)" @dragend="changeposition(card,list)" :config="getgroupconfig(card)">
            <v-rect :config="getrectconfig(card)" ></v-rect>
            <v-text :config="gettextconfig(card)" @dblclick="edittext(card, list, index)" :ref="'text' + card.id"></v-text>
             <v-group>
              <v-tag :config="getcategoryconfig(card)"></v-tag>
              <v-text :config="getcategorytextconfig(list)"></v-text>
            </v-group>
            <v-group v-if="card.showexplain && card.desc.explain !== ''">
              <v-rect :config="getexplainrectconfig(card)"></v-rect>  
              <v-text :config="getexplainconfig(card)"></v-text>
            </v-group>
            <v-group v-for="(person,i) in card.tagsfrom" :key="person">
              <v-tag :config="gettagconfig(person,i,'#ABEBC6')"></v-tag>
              <v-text :config="gettagtextconfig(person,i)"></v-text>
            </v-group>
            <v-group v-for="(person,i) in card.tagsto" :key="person">
              <v-tag :config="gettagconfig(person,i,'#D2B4DE')"></v-tag>
              <v-text :config="gettagtextconfig(person,i)"></v-text>
            </v-group>
            <v-group v-for="(attachment, j) in card.attachments" :key="attachment.name" @mouseup="viewattachment(attachment)" @mouseover="changecursor(true)" @mouseout="changecursor(false)">
              <v-tag :config="gettagconfig(attachment.name, j,'#CFD8DC' , card.tags)"></v-tag>
              <v-text :config="gettagtextconfig(attachment.name, j, card.tags)"></v-text>
            </v-group>
          </v-group>
        </v-group>
        <v-arrow v-for="arrow in arrows" :key="arrow" ref="arrow" :config="arrowconfig[arrow]"></v-arrow>
      </v-layer>
    </v-stage>
    <form-card></form-card>
    <!-- <v-dialog v-model="dialog" persistent max-width="50vw">
      <v-card>
        <v-form ref="form" v-model="valid" lazy-validation @submit.prevent="submit">
          <v-container>
            <v-layout row wrap class="ma-5">
              <v-flex flex xs12 class="pt-5">
                <v-select v-model="newcard.typeid" :items="lists" label="卡片類型" item-text="name" item-value="id" @change="changetype"></v-select>
                <v-layout row wrap v-if="newcard.type == '問題面向'">
                   <v-flex flex xs12>
                    <v-text-field color="blue-grey darken-2" label="內容" v-model="card.title" :counter="30" :rules="titleRules"></v-text-field>
                  </v-flex>
                </v-layout>
                <v-layout row wrap v-if="newcard.type == '問題細節'">
                  <v-flex flex xs12>
                    <v-text-field color="blue-grey darken-2" label="內容"  v-model="card.title" :counter="30" :rules="titleRules"></v-text-field>
                  </v-flex>
                  <v-flex flex xs12>
                    <v-text-field color="blue-grey darken-2" label="補充說明" v-model="card.desc.explain" ></v-text-field>
                  </v-flex>
                </v-layout >
                <v-layout row wrap  v-if="newcard.type == '現有解法'">
                  <v-flex flex xs12>
                    <v-text-field color="blue-grey darken-2" label="內容"  v-model="card.title" :counter="30" :rules="titleRules"></v-text-field>
                  </v-flex>
                </v-layout>
                <v-layout row wrap  v-if="newcard.type == '政府回應'">
                  <v-flex flex xs12>
                    <v-text-field color="blue-grey darken-2" label="內容"  v-model="card.title" :counter="30" :rules="titleRules"></v-text-field>
                  </v-flex>
                  <v-flex flex xs12>
                    <v-radio-group v-model="card.desc.responsetime" row @change="changeresponsetime(card)">
                      <v-radio label="現在" value="nowadays" selected ></v-radio>
                      <v-radio label="未來" color="orange" value="future"></v-radio>
                    </v-radio-group>
                  </v-flex>
                  <v-flex flex xs12>
                    <v-text-field color="blue-grey darken-2" label="補充說明"  v-model="card.desc.explain"  ></v-text-field>
                  </v-flex>
                </v-layout>
                <v-layout row wrap v-if="newcard.type == '困難'">
                  <v-flex flex xs12>
                    <v-text-field color="blue-grey darken-2" label="內容"   v-model="card.title" :counter="30" :rules="titleRules"></v-text-field>
                  </v-flex>
                  <v-flex flex xs12>
                    <v-text-field color="blue-grey darken-2" label="補充說明"  v-model="card.desc.explain"  ></v-text-field>
                  </v-flex>
                </v-layout>
              </v-flex>
              <v-flex md12 v-if="(board.admin.includes(user.id) || board.members.includes(user.id)) && card.title != ''">
                <v-layout row wrap v-if="selectedlist.name != '佐證文件' && selectedlist.name != '利害關係人'">
                  <v-flex>
                    <v-btn flat color="primary" v-if="newpersonmode == false && newattachmentmode == false" @click.native="newpersonmode = true">+新增利害關係人</v-btn>
                    <v-btn flat color="primary" v-if="newattachmentmode == false && newpersonmode == false" @click.native="newattachmentmode = true">+新增佐證文件
                      <v-tooltip bottom>
                        <v-icon slot="activator" class="ml-1">info</v-icon>
                        <span>1.既有資料是否可以先提供？這些資料包含：<br/>
                        -過去各項會議(內部會議、跨部會協調會議、專家學者會議等)討論的相關內參資料<br/>
                        -是否有推薦相關的研究論文<br/>
                        -其他不同意立場的陳述或是好文章<br/>
                        2.相關法制規定為何？過去相關規定的修正有沒有討論記錄？<br/>
                        3.有沒有立委曾經關心過此議題？立法院有沒有討論過此議題？有沒有會議記錄與部會當時的文書？<br/>
                        4.有沒有民間關心過此議題?<br/>
                        -有沒有NGO關心過此議題？有沒有相關的資料？<br/>
                        -有沒有陳抗過？收過陳情書？<br/>
                        -部長信箱有沒有收過類似的問題？<br/>
                        </span>
                      </v-tooltip>
                    </v-btn>
                  </v-flex>
                </v-layout>
              </v-flex>
              <v-flex md12 v-if="newpersonmode == true">
                <v-layout row wrap>
                  <v-flex md6>
                    <v-text-field color="blue-grey darken-2" label="稱謂/單位名稱"  v-model="newperson.title" ></v-text-field>
                  </v-flex>
                  <v-flex md6>
                    <v-btn color="black" class="mt-3" small outline @click.native="newpersonmode = false; newperson.title = ''" >取消</v-btn>
                    <v-btn color="black" class="mt-3" small dark @click.native="newpersonmode = false; addperson()">確定</v-btn>
                  </v-flex>
                </v-layout>
              </v-flex>
              <v-flex flex md12 v-if="newpersonmode == false && newattachmentmode == false && card.title != ''">
                <v-layout row wrap v-if="selectedlist.name != '利害關係人'">
                  <v-flex flex xs12 >
                    <v-select
                      v-model="card.desc.stakeholders"
                      :items="peoplelist"
                      item-text="name"
                      item-value="id"
                      label="關聯利害關係人(資料來源)"                     
                      chips
                      multiple
                      deletable-chips
                      no-data-text="目前尚無資料"
                    >
                      <template slot="item" slot-scope="data">
                        <template v-if="typeof data.item !== 'object'">                   
                        </template>
                        <template v-else>
                          <v-list-tile-avatar>
                            <v-checkbox v-model="card.desc.stakeholders" :value="data.item.id"></v-checkbox>
                          </v-list-tile-avatar>
                          <v-list-tile-content v-text="data.item.name"></v-list-tile-content> 
                        </template>
                      </template>                                        
                    </v-select>
                  </v-flex>
                </v-layout>
              </v-flex>
               <v-flex md12 v-if="newattachmentmode == true">
                <v-layout row wrap>
                  <v-flex md12>
                    <v-radio-group v-model="attachmentselection" row >
                      <v-radio label="新增連結" value="attachmentlink"></v-radio>
                      <v-radio label="上傳檔案" value="attachmentupload"></v-radio>
                    </v-radio-group>
                  </v-flex>
                  <v-layout row wrap v-if="attachmentselection == 'attachmentlink'">
                    <v-flex md12>
                      <v-text-field color="blue-grey darken-2" label="佐證文件名稱"  v-model="newattachment.title" :counter="30" :rules="titleRules"></v-text-field>
                    </v-flex>
                    <v-flex md12>
                      <v-text-field color="blue-grey darken-2" label="文件連結"  v-model="newattachment.desc.attachment"></v-text-field>
                    </v-flex>
                  </v-layout>
                  <v-layout row wrap v-if="attachmentselection == 'attachmentupload'">
                    <v-flex md6>
                      <v-text-field color="blue-grey darken-2" label="佐證文件名稱"  v-model="newattachment.title" :counter="30" :rules="titleRules"></v-text-field>
                    </v-flex>
                    <v-flex md4>
                      <upload-btn depressed outline title="附加檔案" :fileChangedCallback="fileChanged" class="mt-3">
                        <template slot="icon-left">
                          <v-icon left>attach_file</v-icon>
                        </template>
                      </upload-btn>
                    </v-flex>
                    <v-flex md2>
                      <h3 class="mt-4">{{filename}}</h3>
                    </v-flex>
                    <v-flex md12>

                    </v-flex>
                  </v-layout>
                  <v-flex md12>
                    <v-layout align-center justify-end row fill-height>
                    <v-btn color="black" class="mt-3" small outline @click.native="newattachmentmode = false; newattachment.title = ''; newattachment.desc.attachment = '';" >取消</v-btn>
                    <v-btn color="black" class="mt-3" small dark @click.native="newattachmentmode = false; addattachment()">確定</v-btn>
                  </v-layout>
                  </v-flex>
                </v-layout>
              </v-flex>
              <v-flex flex md12 v-if="newpersonmode == false && newattachmentmode == false && card.title != ''">
                <v-layout row wrap v-if="selectedlist.name != '佐證文件' && selectedlist.name != '利害關係人'">
                  <v-flex flex xs12 >
                    <v-select
                      v-model="card.desc.evidences"
                      :items="datalist"
                      item-text="name"
                      item-value="id"
                      label="選取佐證文件"
                      color="blue-grey darken-2"
                      chips
                      multiple
                      deletable-chips
                      no-data-text="目前尚無資料"
                    >
                      <template slot="item" slot-scope="data">
                        <v-list-tile-avatar>
                          <v-checkbox v-model="card.desc.evidences" :value="data.item.id"></v-checkbox>
                        </v-list-tile-avatar>
                        <v-list-tile-content v-text="data.item.name"></v-list-tile-content>
                      </template>
                    </v-select>
                  </v-flex>
                </v-layout>
              </v-flex>
            </v-layout>
            <v-card-actions class="pa-3" v-if="board.admin.includes(user.id) || board.members.includes(user.id)">
              <v-btn flat color="grey lighten-1" class="subheading" @click="resetForm">重新填寫</v-btn>
              <v-spacer></v-spacer>
              <v-btn flat @click.native="dialog = false; noteicon = false" class="subheading">取消</v-btn>
              <v-btn flat color="cyan" type="submit" class="subheading" @click.native="noteicon = false">確認</v-btn>
            </v-card-actions>
          </v-container>
        </v-form>
      </v-card>
    </v-dialog> -->
  </div>
</div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import { createHelpers } from 'vuex-map-fields';
import card from './forms/card'
import UploadButton from 'vuetify-upload-button';

const { mapFields: mapBoardFields } = createHelpers({
  getterType: 'getBoardField',
  mutationType: 'updateBoardField',
});

const { mapFields: mapCardFields } = createHelpers({
  getterType: 'getCardField',
  mutationType: 'updateCardField',
});

const { mapFields: mapListFields } = createHelpers({
  getterType: 'getListField',
  mutationType: 'updateListField',
});

export default {
  components: {
    'form-card': card,
    'upload-btn': UploadButton
  },
  data() {
    return {
      arrows: 0,
      arrowconfig: [],
      labeltextconfig: [],
      dialog: false,
      valid: false,
      selectedlist: {},
      peoplelist: [],
      datalist: [],
      snackbar: false,
      textbox: {
        text: '',
        width: '180px',
        height: '130px',
        show: false,
      },
      editdialog: false,
      titleRules: [
        v => !!v || '此欄位為必填!',
        v => v.length <= 30 || '此欄位不可超過30個字!'
      ],
      pressshift: false,
      firstcard: {},
      newcard: {
        typeid: '',
        type: '',
        title: '',
        desc:{
          explain:'',
          responsetime: 'nowadays',
          department: '',
          background: '',
          role: '',
          stakeholders: [],
          evidences: [],
          related: [],
          attachment: '',
          x: 0,
          y: 0
        },
      },
      responsestring: '',
      isexplain: false,
      noteicon: false,
      linkicon: false,
      unlinkicon: false,
      tab: 'tab-1',
      newpersonmode: false,
      newattachmentmode: false,
      attachmentselection: 'attachmentlink',
      newattachment: {
        title: '',
        desc: {
          explain:'',
          responsetime: 'nowadays',
          department: '',
          background: '',
          role: '',
          stakeholders: [],
          evidences: [],
          related: [],
          attachment: '',
          x: 0,
          y: 0
        }
      },
      filename: '',
      newperson: {
        title: '',
        desc: {
          explain:'',
          responsetime: 'nowadays',
          department: '',
          background: '',
          role: '',
          stakeholders: [],
          evidences: [],
          related: [],
          attachment: '',
          x: 0,
          y: 0
        }
      },
    }
  },
  created: function() {
    this.$store.dispatch('getlists', this.$route.params.id)
    /* this.getcards() */
    /* this.getlabeltextconfig() */
    this.getboard()
  },
  mounted: function() {
    let that = this
    let stage = this.$refs.stage.getStage()
    setTimeout( () => {
      this.getarrows()
      let rects = stage.find('.rectgroup')
      rects.each( r => {
        r.setZIndex(5)
        stage.draw()
      })
    }, 1000)
    window.addEventListener('wheel', (e) => {
      let stage = this.$refs.stage.getStage()
      let scaleBy = 1.1;
      e.preventDefault();
      let oldScale = stage.scaleX();

      let mousePointTo = {
          x: stage.getPointerPosition().x / oldScale - stage.x() / oldScale,
          y: stage.getPointerPosition().y / oldScale - stage.y() / oldScale,
      };

      let newScale = e.deltaY < 0 ? oldScale * scaleBy : oldScale / scaleBy;
      stage.scale({ x: newScale, y: newScale });

      let newPos = {
          x: -(mousePointTo.x - stage.getPointerPosition().x / newScale) * newScale,
          y: -(mousePointTo.y - stage.getPointerPosition().y / newScale) * newScale
      };
      stage.position(newPos);
      stage.batchDraw();
    });
    window.addEventListener("keydown", (e) => {
      if (event.keyCode == 16) {
        this.pressshift = true
      }
    });
    window.addEventListener("keyup", (e) => {
      if (event.keyCode == 16) {
        this.pressshift = false
      }
    });
  },
  destroyed: function() {
    window.removeEventListener('wheel', (e) => {
      e.preventDefault();
      let oldScale = stage.scaleX();

      let mousePointTo = {
          x: stage.getPointerPosition().x / oldScale - stage.x() / oldScale,
          y: stage.getPointerPosition().y / oldScale - stage.y() / oldScale,
      };

      let newScale = e.deltaY > 0 ? oldScale * scaleBy : oldScale / scaleBy;
      stage.scale({ x: newScale, y: newScale });

      let newPos = {
          x: -(mousePointTo.x - stage.getPointerPosition().x / newScale) * newScale,
          y: -(mousePointTo.y - stage.getPointerPosition().y / newScale) * newScale
      };
      stage.position(newPos);
      stage.batchDraw();}
    )
  },
  methods: {
    /* getcards: async function() {
      this.lists = []
      let id = this.$route.params.id
      let listarray = await Trello.boards.get(id + '/lists',{cards: 'open'})
      listarray.map( await (l => {
        if (l.name != '利害關係人' && l.name != '佐證文件') {
          let list = {}
          list.id = l.id
          list.name = l.name
          list.cards = l.cards
          switch (list.name)
          {
            case '問題面向':
            list.color = '#FFCD13'
            list.column = 1
            break
            case '問題細節':
            list.color = '#FFE276'
            list.column = 2
            break
            case '現有解法':
            list.color = '#91AD70'
            list.column = 3
            break
            case '政府回應':
            list.color = '#F08B8B'
            list.column = 4
            break
            case '困難':
            list.color = '#C85938'
            list.column = 5
            break
            case '利害關係人':
            list.color = '#0097A7'
            list.column = 6
            break
            case '佐證文件':
            list.color = '#CFD8DC'
            list.column = 7
            break
            default:
            list.color = 'teal'
            break
          }
          list.cards.map( async (card) => {
            let desc = JSON.parse(card.desc)
            card.desc = desc
            card.color = list.color
            card.column = list.column
            card.showexplain = false
            card.hover = false
            card.tagsfrom = []
            card.tagsto = []
            card.desc.stakeholders.map( async (personid) => {
              let person = await Trello.cards.get(personid)
              card.tagsfrom.push(person.name)
            })
            card.attachments = []
            card.desc.evidences.map( async (attachid) => {
              let attach = await Trello.cards.get(attachid,{fields: 'attachments',attachments: true})
              if (attach.attachments.length != 0) {
                attach.attachments.map( async (att) => {
                  let attachment = {}
                  attachment.name = att.name
                  attachment.url = att.url
                  card.attachments.push(attachment)
                })
              }
            })
          })
          this.lists.push(list)
        }
      }))
    }, */
    getarrows: function() {
      let that = this
      this.arrows = 0
      that.lists.map(l => {
        l.cards.map(c => {
          if (c.desc.related != undefined) {
            if (c.desc.related.length != 0) {
              c.desc.related.map( r => {
                that.arrows++
                that.getarrowconfig(c.id,r)
              })
            }
          }
        })
      })
    },
    getstageconfig: function() {
      let stagewidth = window.innerWidth
      let stageheight = window.innerHeight - 64
      return {
        width: stagewidth,
        height: stageheight,
        draggable: true
      }
    },
    getgroupconfig: function(card) {
      return {
        x: card.desc.x,
        y: card.desc.y,
        draggable: true,
        name: 'rectgroup'
      }
    },
    getrectconfig: function(card) {
      console.log(card.desc.stakeholders)
      let tags = 0/* card.desc.stakeholders.length + card.desc.evidences.length */
      if (tags < 2) { tags = 2 }
      return {
        fill: "#FBF0D3",
        width: 160,
        height: 130 + ((tags - 2) * 20),
        shadowColor: 'black',
        shadowOffset: {
          x: 5,
          y: 5
        },
        shadowOpacity: 0.5
      }
    },
    gettextconfig: function(card) {
      return {
        x: 0,
        y: 15, 
        text: card.name, 
        fontSize: 18, 
        width: 160, 
        padding: 15, 
        fontFamily:  "Roboto, 'Noto Sans TC'" ,
        lineHeight: 1.5,
      }
    },
    getcategoryconfig: function(card) {
      return {
        x: 0,
        y: 10,
        fill: card.categorycolor,
        width: 70,
        height: 20,
        lineJoin: 'round',
      }
    },
    getcategorytextconfig: function(list) {
      return {
        x: 5,
        y: 8,
        width: 70,
        text: list.name,
        fontSize: 12,
        fontFamily: "Roboto, 'Noto Sans TC'",
        padding: 5,
      }
    },
    gettagconfig: function(text, i, color, tags) {
      let textlength = text.length
      let tagslength = 0
      let j = 0
      let k = 0
      if (tags != undefined) {
        tagslength = tags.length
        if (tagslength + i > 1) {
          j = (tagslength + i) / 2
        }
        k = (tagslength + i) % 2
      }
      else {
        k = (tagslength + i) % 2
      }
      return {
        x: 20 + k * 75,
        y: 100 + j * 30,
        width: 60,
        height: 20,
        fill: color,
        pointerDirection: 'left',
        pointerWidth: 10,
        pointerHeight: 20,
        lineJoin: 'round',
      }
    },
    gettagtextconfig: function(text, i, tags) {
      let textlength = text.length
      let tagslength = 0
      let j = 0
      let k = 0
      if (tags != undefined) {
        tagslength = tags.length
        if (tagslength + i > 1) {
          j = (tagslength + i) / 2
        }
        k = (tagslength + i) % 2
      }
      else {
        k = (tagslength + i) % 2
      }
      return {
        x: 25 + k * 75,
        y: 103 + j * 30,
        width: textlength * 15,
        text: text,
        fontSize: 12,
        fontFamily: "Roboto, 'Noto Sans TC'",
      }
    },
    getarrowconfig: function(c,r) {
      let startpoint = this.$refs[c][0].getStage()
      let endpoint = this.$refs[r][0].getStage()
      let startpointx = startpoint.getX() + startpoint.children[0].getWidth()/2
      let startpointy = startpoint.getY() + startpoint.children[0].getHeight()
      let endpointx = endpoint.getX() + endpoint.children[0].getWidth()/2
      let endpointy = endpoint.getY()
      this.arrowconfig[this.arrows] = {
        name: c + ',' + r,
        fill: 'black',
        points: [startpointx,startpointy,endpointx,endpointy],
        stroke: 'black',
        strokeWidth: 4,
        shadowColor: 'black',
        shadowOffset: {
          x: 1,
          y: 1
        },
        shadowOpacity: 0.5,
      }
    },
    adjustPoint: function(id){
      let point = this.$refs[id][0].getStage()
      this.$refs.arrow.map(a => {
        let ids = a.getStage().getName().split(',')
        if (id == ids[0]) {
          let p = [point.getX() + point.children[0].getWidth()/2,point.getY() + point.children[0].getHeight(),a.getStage().getPoints()[2],a.getStage().getPoints()[3]]
          a.getStage().setPoints(p)
        } else if (id == ids[1]) {
          let p = [a.getStage().getPoints()[0],a.getStage().getPoints()[1],point.getX() + point.children[0].getWidth()/2,point.getY()]
          a.getStage().setPoints(p)
        }       
      })
    },
    changeposition: function(card,list) {
      card.desc.x = this.$refs[card.id][0].getStage().getX()
      card.desc.y = this.$refs[card.id][0].getStage().getY()
      Trello.put('cards/' + card.id, {'idList': list.id,'desc': JSON.stringify(card.desc) } , function() {
      })
    },
    viewattachment: function(attachment) {
      window.open(attachment.url);
    },
    changecursor: function(mouseover) {
      /* if (mouseover) {
        document.body.style.cursor = 'pointer';
      }
      else {
        document.body.style.cursor = 'default';
      } */
    },
    linkcard: function(card) {
      let that = this
      if (this.pressshift) {
        if (Object.keys(this.firstcard).length == 0) {
          this.firstcard = card
          this.$refs[card.id][0].getStage().children[0].fill('black')
          this.$refs[card.id][0].getStage().children[1].fill('white')
          this.$refs.stage.getStage().draw()
        }
        else {
          if (this.firstcard.column == 1) {
            if (this.firstcard.column == card.column || this.firstcard.column + 1 == card.column) {
              if (!this.firstcard.desc.related.includes(card.id) && this.firstcard.id !== card.id) {
                this.firstcard.desc.related.push(card.id)
                Trello.put('cards/' + this.firstcard.id, {'desc': JSON.stringify(this.firstcard.desc) } , function() {
                  that.firstcard = {}
                  that.getcards()
                  that.editdialog = false
                  setTimeout( () => {
                    that.getarrows()
                  }, 1000)
                })
              } else {
                let index = this.firstcard.desc.related.indexOf(card.id);
                if (index !== -1) this.firstcard.desc.related.splice(index, 1);
                Trello.put('cards/' + this.firstcard.id, {'desc': JSON.stringify(this.firstcard.desc) } , function() {
                  that.firstcard = {}
                  that.getcards()
                  that.editdialog = false
                  setTimeout( () => {
                    that.getarrows()
                  }, 1000)
                })
              }
            } else {
              this.firstcard = {}
              this.getcards()
            }
          } else {
            if (this.firstcard.column + 1 == card.column) {
              if (!this.firstcard.desc.related.includes(card.id) && this.firstcard.id !== card.id) {
                this.firstcard.desc.related.push(card.id)
                Trello.put('cards/' + this.firstcard.id, {'desc': JSON.stringify(this.firstcard.desc) } , function() {
                  that.firstcard = {}
                  that.getcards()
                  that.editdialog = false
                  setTimeout( () => {
                    that.getarrows()
                  }, 1000)
                })
              } else {
                let index = this.firstcard.desc.related.indexOf(card.id);
                if (index !== -1) this.firstcard.desc.related.splice(index, 1);
                Trello.put('cards/' + this.firstcard.id, {'desc': JSON.stringify(this.firstcard.desc) } , function() {
                  that.firstcard = {}
                  that.getcards()
                  that.editdialog = false
                  setTimeout( () => {
                    that.getarrows()
                  }, 1000)
                })
              }
            } else {
              this.firstcard = {}
              this.getcards()
            }
          }
        }
      }
      /* let stage = this.$refs.stage.getStage()
      stage.addEventListener('keydown', function (e) {
         if (e.keyCode === 37) {
         }
      }) */
    },
    edittext: function(card, list, index) {
      this.dialog = true
      this.newcard.type = list.name;
      this.newcard.typeid = list.id;
     /*  this.selectedlist.color = list.color
      this.selectedlist.cards = list.cards
      this.selectedlist.column = list.column */
      this.card.id = card.id
      this.card.title = card.name
      this.card.desc.responsetime = card.desc.responsetime
      this.card.desc.stakeholders = card.desc.stakeholders
      this.card.desc.evidences = card.desc.evidences
      this.card.desc.related = card.desc.related
      this.card.desc.explain = card.desc.explain
      this.card.desc.role = card.desc.role
      this.card.desc.department = card.desc.department
      this.card.desc.background = card.desc.background
      this.card.desc.attachment = card.desc.attachment 
      this.card.desc.x = card.desc.x
      this.card.desc.y = card.desc.y
      this.card.attachments = card.attachments
      
      /* let x = this.$refs[card.id][0].getStage().getX()
      let y = this.$refs[card.id][0].getStage().getY()
      let textarea = document.getElementById("textbox");
      let position = this.$refs[card.id][0].getStage().getAbsolutePosition()
      let left = this.$refs.stage.getStage().getContainer().getBoundingClientRect().left
      let top = this.$refs.stage.getStage().getContainer().getBoundingClientRect().top
      let areaPosition = {
        x: position.x + left ,
        y: position.y + top
      };
      console.log(areaPosition)
      var textarea = document.createElement('textarea');
      document.body.appendChild(textarea);
      document.getElementById("stage").appendChild(textarea);
      textarea.value = card.name;
      textarea.style.position = 'absolute';
      textarea.style.left = areaPosition.x + 'px';
      textarea.style.top = areaPosition.y + 'px';
      textarea.style.width = '180px';
      textarea.focus();
      console.log(areaPosition.y + 'px') */
    },
    edittextsubmit: function() {
      let that = this
      Trello.put('cards/' + this.card.id, {'name': this.card.title } , function(res) {
        that.getcards()
        that.editdialog = false
        setTimeout( () => {
          that.getarrows()
        }, 1000)
        /* let text = that.$refs['text' + this.card.id][0].getStage()
        text.text(that.card.title)
        let stage = that.$refs.stage.getStage();
        stage.draw() */
      })
    },
    changetype: function() {
      this.lists.map( list => {
        if (list.id == this.newcard.typeid) {
          this.newcard.type = list.name
          this.resetForm()
          if (this.newcard.type == '政府回應') {
            if (this.newcard.desc.responsetime == 'nowadays') {
              this.newcard.title = '[現在]'
            } else {
              this.newcard.title = '[未來]'
            }
          }
        }
      })
    },
    getboard: function() {
      let that = this;
      this.board.id = this.$route.params.id
      Trello.boards.get(this.board.id,{'fields':'all'}, function(res) {
        that.board.name = res.name
        if (res.desc != '') {
          that.board.desc = JSON.parse(res.desc)
        }
        that.board.admin = []
        that.board.members = []
        res.memberships.map( m => {
          if (m.memberType == 'admin') {
            that.board.admin.push(m.idMember)
          }
          else {
            that.board.members.push(m.idMember)
          }
        })
      })
    },
    resetForm: function() {
      this.newcard.title = ''
      this.newcard.desc.explain = ''
      this.newcard.desc.department= ''
      this.newcard.desc.background= ''
      this.newcard.desc.role= ''
      this.newcard.desc.evidences = []
    },
    changeresponsetime: function(card) {
      card.title = card.title.replace('[現在]','').replace('[未來]','')
      if (card.desc.responsetime == 'nowadays') {
        this.responsestring = '[現在]'
      } else {
        this.responsestring = '[未來]'
      }
      card.title = this.responsestring + card.title
    },
    getexplainrectconfig: function(card) {
      return {
        fill: 'grey',
        width: 180,
        height: 130,
        shadowColor: 'black',
        shadowOffset: {
          x: 5,
          y: 5
        },
        shadowOpacity: 0.5
      }
    },
    getexplainconfig: function(card) {
      return { 
        text: card.desc.explain, 
        fill: 'pink',
        fontSize: 18, 
        width: 180, 
        padding: 15, 
        fontFamily:  "Roboto, 'Noto Sans TC'" ,
        lineHeight: 1.5,
        draggable: true
      }
    },
    showexplain: function(card, show) {
      card.showexplain = show
    },
    changeiconcolor: function(icon) {
      if (icon == 'black') {
        icon = 'blue'
      } else {
        icon = 'black'
      }
    },
    submit: function() {
      let that = this
      this.newcard.desc.x = 600
      this.newcard.desc.y = 400
      Trello.post('cards', {'name': this.newcard.title, 'idList': this.newcard.typeid,'desc': JSON.stringify(this.newcard.desc)} , function(res) {
        that.dialog = false
        that.getcards()
      })
    },
    bindtitlestyle: function(title) {
      if (title == '問題面向') {
        this.titlecolor = '#FFCD13'
      } else if (title == '問題細節') { 
        this.titlecolor = '#FFE276'
      } else if (title == '現有解法') { 
        this.titlecolor = '#91AD70'
      } else if (title == '政府回應') { 
        this.titlecolor = '#F08B8B'
      } else if (title == '困難') { 
        this.titlecolor = '#C85938'
      } else if (title == '利害關係人') { 
        this.titlecolor = '#21B5C2'
      } else if (title == '佐證文件') { 
        this.titlecolor = '#D8CAC4'
      } 
      return this.titlestyle + this.titlecolor
    },
    fileChanged: function(file) {
      let formData = new FormData();
      formData.append('key','fb8dab318e1888679f571104d8b36ac7')
      formData.append('token',localStorage.trello_token)
      formData.append("file", file)
      formData.append("name", this.newattachment.title);
      this.filename = file.name
      this.uploadfile = formData
    },
    /* handlescroll: function() {
      console.log(window.scrollY)
      let stage = this.$refs.stage.getStage()
      let scaleBy = 1.1;
      let oldScale = stage.scaleX();
      let mousePointTo = {
          x: stage.getPointerPosition().x / oldScale - stage.x() / oldScale,
          y: stage.getPointerPosition().y / oldScale - stage.y() / oldScale,
      };
      let newScale = window.scrollY.deltaY > 0 ? oldScale * scaleBy : oldScale / scaleBy;
      stage.scale({ x: newScale, y: newScale });

      let newPos = {
          x: -(mousePointTo.x - stage.getPointerPosition().x / newScale) * newScale,
          y: -(mousePointTo.y - stage.getPointerPosition().y / newScale) * newScale
      };
      stage.position(newPos);
      stage.batchDraw();
    } */
  },
  computed: {
    ...mapGetters({
      user: 'user',
    }),
    ...mapBoardFields({
      board: 'board',
    }),
    ...mapCardFields({
      card: 'card',
      opencard: 'opencard',
      selectedlist: 'selectedlist',
      editable: 'editable',
      titlestyle: 'titlestyle',
      titlecolor: 'titlecolor',
      relatedlist: 'relatedlist'
    }),
    ...mapListFields({
      lists: 'lists',
    }),
  }
}
</script>

<style scoped>
body {
  overflow: hidden;
}

</style>
