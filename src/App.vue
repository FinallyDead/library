<template>
<div class="row mb-5 align-items-start">
  <div class="col-sm">
    <img src="./assets/logoFinal.png" class="logo" alt="Responsive image">
  </div>
  <div class="col-sm">
    <input type="file" class="form-control inputFile" @change="onChange" />
    <div style="margin-top: 5px;">
      <div>
        Количество записей: {{ this.collections.length }}
      </div>
      <div>
        Количество страниц: {{ this.totalPages }}  
      </div>
      <div>
        <label class="d-inline">Количество записей на одной странице:</label>  
        <input 
        class="form-control d-inline ms-2" 
        style="width: 15%;"
        type="text" 
        :placeholder="this.limit" 
        @keydown.enter="this.limit = $event.target.value"
        @keyup.enter="$event.target.value = '' "
        />
      </div>
      <div v-if="collections.length > 0">
        <label class="d-inline">Сортировка по:</label>
        <select class="form-select d-inline w-auto ms-2" v-model="sortOption" @change="sortBooks">
          <option selected value="0"> Отсутствует </option>
          <option
          v-for="keyFromArr in selectValues" 
          :key="keyFromArr" 
          :value="keyFromArr"
          >{{ keyFromArr }}</option>
        </select>
      </div>
      <div v-if="collections.length > 0" class="mt-2">
        <label>Поиск по столбцу</label>
        <input 
        v-model="searchQuery"
        class="form-control d-inline ms-2"
        style="width: 30%;" 
        placeholder="Поиск по..."
        @input="search"
        />
        <select v-model="searchOption" class="form-select d-inline w-auto ms-2">
          <option 
          v-for="str in keyNames" 
          :key="str"
          :value="str"
          > {{ str }}</option>
        </select>
      </div>
      <div v-if="this.collectionsPage.length > 0">
        <button class="_btn" @click="exportToExcel('xlsx')"> Сохранить таблицу </button>
      </div>
    </div>
  </div>
</div>
<form class="form" v-show="this.selectedRow" @submit.prevent>
  <input class="form-control input" type="text" placeholder="Автор(-ы)" v-model="this.inputAuthor" />
  <input class="form-control input" type="text" placeholder="Название" v-model="this.inputName" />
  <input class="form-control input" type="text" placeholder="Шкаф" v-model="this.inputCloset" />
  <input class="form-control input" type="text" placeholder="Год" v-model="this.inputYear" />
  <input class="form-control input" type="text" placeholder="Тема" v-model="this.inputThemen" />
  <input class="form-control input" type="text" placeholder="Категория" v-model="this.inputCategory" />
  <div style="margin-top: 15px; align-self: flex-end;">
    <button class="btn_in_div d-inline" style="margin-right: 15px;" @click="changeIntoTable">Изменить</button>
    <button class="btn_in_div d-inline" @click="addIntoTable">Добавить</button>
  </div>
  <button class="_btn" @click="removeFromTable">Удалить</button>
</form>
<div class="page__wrapper" v-if="totalPages > 1">
  <div 
  v-if="this.page > 1"
  class="page"
  @click="changeToEdgeOfPages('lower')"
  >
    &lt;&lt;&lt;
  </div>
  <div
  v-if="this.page > 1"
  class="page"
  @click="this.changePageWithButton('lower')"
  >
    &lt;&lt;
  </div>
  <div 
  v-for="pageNumber in totalPages"
  :key="pageNumber"
  :value='pageNumber * this.limit'
  @click="event => changePage(event, pageNumber)"
  >
    <div
    v-if="!((pageNumber > page+3) || (page-3 > pageNumber)) 
    && totalPages > 20 
    || (page === totalPages && pageNumber > page - 7)
    || (page === totalPages-1 && pageNumber > page - 6)
    || (page === totalPages-2 && pageNumber > page - 5)
    || (page === 1 && pageNumber < page + 7)
    || (page === 2 && pageNumber < page + 6)
    || (page === 3 && pageNumber < page + 5)"
    class="page"
    :class="{
      'current-page': page === pageNumber
    }"
    >
      {{ pageNumber }}
    </div>
    <div 
    v-else-if="totalPages < 20"
    class="page"
    :class="{
      'current-page': page === pageNumber
    }" 
    >
    {{ pageNumber }}
    </div>
</div>
  <div
  v-if="this.page < this.totalPages"
  class="page"
  @click="this.changePageWithButton('higher')"
  >
    >>
  </div>
  <div 
  v-if="this.page < this.totalPages"
  class="page"
  @click="changeToEdgeOfPages('higher')"
  >
    >>>
  </div>
</div> 
 <section v-if="collectionsPage.length > 0" class="tableSec">
    <table id="tbl_exporttable_to_xls" class="table table-bordered table-striped">
      <thead>
        <tr>
          <th 
          v-for="header in keyNames" 
          :key="header"
          >
            {{ header }}
          </th>
        </tr>  
      </thead>
      <tbody >
        <tr 
        v-for="strings in collectionsPage" 
        :key="strings.id"
        @click="selectRow"
        :class="{
          'activeItem': strings.id === this.selectedRow
        }"
        >
          <td 
          v-for="(item,key) in strings" 
          :key="key" 
          :value="item"
          >
            {{ item }}
          </td>
        </tr>
      </tbody>
    </table> 
  </section>
  <div class="waitLabel" v-if="collections.length == 0">
      Выберите файл для отображения
  </div>
  <div class="waitLabel" v-if="collections.length > 0 && collectionsPage.length == 0">
      Ничего не найдено
  </div>
  <div class="page__wrapper" v-if="totalPages > 1">
    <div 
    v-if="this.page > 1"
    class="page"
    @click="changeToEdgeOfPages('lower')"
    >
      &lt;&lt;&lt;
    </div>
    <div
    v-if="this.page > 1"
    class="page"
    @click="this.changePageWithButton('lower')"
    >
      &lt;&lt;
    </div>
    <div 
    v-for="pageNumber in totalPages"
    :key="pageNumber"
    :value='pageNumber * this.limit'
    @click="event => changePage(event, pageNumber)"
    >
      <div
      v-if="!((pageNumber > page+3) || (page-3 > pageNumber)) 
      && totalPages > 20 
      || (page === totalPages && pageNumber > page - 7)
      || (page === totalPages-1 && pageNumber > page - 6)
      || (page === totalPages-2 && pageNumber > page - 5)
      || (page === 1 && pageNumber < page + 7)
      || (page === 2 && pageNumber < page + 6)
      || (page === 3 && pageNumber < page + 5)"
      class="page"
      :class="{
        'current-page': page === pageNumber
      }"
      >
        {{ pageNumber }}
      </div>
      <div 
      v-else-if="totalPages < 20"
      class="page"
      :class="{
        'current-page': page === pageNumber
      }" 
      >
      {{ pageNumber }}
      </div>
  </div>
    <div
    v-if="this.page < this.totalPages"
    class="page"
    @click="this.changePageWithButton('higher')"
    >
      >>
    </div>
    <div 
    v-if="this.page < this.totalPages"
    class="page"
    @click="changeToEdgeOfPages('higher')"
    >
      >>>
    </div>
  </div>
</template>

<script>
import * as XLSX from "xlsx";
import "bootstrap/dist/css/bootstrap.min.css";

export default {
  data() {
    return {
      file: null,
      keyNames: [],
      collections: [],
      collectionsPage: [],
      currentIndex: 0,
      pastIndex: 0,
      selectedRow: null,
      page: 1,
      limit: 25,
      totalPages: 0,
      pagesPerBlock: 10,
      inputAuthor: "",
      inputName: "",
      inputCloset: "",
      inputYear: "",
      inputThemen: "",
      inputCategory: "",
      sortOption: 0,
      selectValues: [],
      searchQuery: "",
      searchOption: -1,
    };
  },
  methods: {
    onChange(event) {
      this.file = event.target.files ? event.target.files[0] : null;
      const reader = new FileReader();
      reader.onload = (e) => {
              
              const bstr = e.target.result;
              const wb = XLSX.read(bstr, { type: "binary" });
              
              const wsname = wb.SheetNames[0];
              const ws = XLSX.utils.sheet_to_row_object_array(wb.Sheets[wsname]);
              
              this.collections = JSON.parse(JSON.stringify(ws));
              this.keyNames = Object.keys(this.collections[0]);
              this.selectValues = this.keyNames.slice(1);
              this.totalPages = Math.ceil(this.collections.length / this.limit);
              this.currentIndex = this.limit;
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            };

            reader.readAsBinaryString(this.file);
            this.file=null;
            
    },
    selectRow(e){
      this.selectedRow = parseInt(e.currentTarget.querySelector('td').innerText);
    },
    changeIndexOfPage(targetValue){
      if(this.currentIndex < targetValue){
        this.pastIndex = targetValue - this.limit;
        this.currentIndex = parseInt(targetValue);
        if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
        this.selectedRow = null;
      }
      else{
        this.currentIndex = parseInt(targetValue);
        this.pastIndex = this.currentIndex - this.limit;
        if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
        this.selectedRow = null;
      }
    },
    changePage(e, pageNumber){
        this.page = pageNumber;
        let targetValue = e.currentTarget.getAttribute("value");
        this.changeIndexOfPage(targetValue);
    },
    changePageWithButton(side){
      switch(side){
          case "lower":
            --this.page;
            this.currentIndex = this.pastIndex;
            this.pastIndex = this.currentIndex - this.limit;
            if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
            this.selectedRow = null;
            break;
          case "higher":
            ++this.page;
            this.pastIndex = parseInt(this.currentIndex);
            this.currentIndex = parseInt(this.currentIndex) + this.limit;
            if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
            this.selectedRow = null;
            break;  
        }
    },
    changeToEdgeOfPages(side){
        switch(side){
          case "lower":
            this.page = 1;
            this.pastIndex = 0;
            this.currentIndex = this.page * this.limit;
            if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
            this.selectedRow = null;
            break;
          case "higher":
            this.page = this.totalPages;
            this.currentIndex = this.page * this.limit;
            this.pastIndex = this.currentIndex - this.limit;
            if (this.sortOption == 0 && this.searchQuery == "")
              this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
            else if (this.searchQuery != "" && this.searchOption != -1)
                this.search();
              else
                this.sortBooks();
            this.selectedRow = null;
            break;  
        }
    },
    addIntoTable(){
      let arrPart = this.collections.splice(this.collections.indexOf(this.collections.find(x => x.id == this.selectedRow))+1, 
      this.collections.length);
      this.collections.push({id: this.selectedRow+1 , author: this.inputAuthor, name: this.inputName, 
        closet: this.inputCloset, year: this.inputYear, themen: this.inputThemen, category: this.inputCategory });
      arrPart.forEach(element => { 
        element.id++;
        this.collections.push(element);
      });
      this.selectedRow = null;
      if (this.totalPages < Math.ceil(this.collections.length / this.limit)){
        this.page++;
        this.currentIndex + this.limit;
        this.pastIndex + this.limit;
      }
      this.inputAuthor = "";
      this.inputName = "";
      this.inputCloset = "";
      this.inputYear = "";
      this.inputThemen = "";
      this.inputCategory = "";
      this.totalPages = Math.ceil(this.collections.length / this.limit);
      if (this.sortOption == 0)
        this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
      else 
        this.sortBooks();
    },
    removeFromTable(){
      let arrPart = this.collections.splice(this.collections.indexOf(this.collections.find(x => x.id == this.selectedRow))+1,
      this.collections.length);
      this.collections.pop();
      arrPart.forEach(element => { 
        element.id--;
        this.collections.push(element);
      });
      this.selectedRow = null;
      this.totalPages = Math.ceil(this.collections.length / this.limit);
      if (this.sortOption == 0){
        if (this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex).length>0){
         this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex)
        }else{
         this.collectionsPage = this.collections.filter(p => p.id<=(this.currentIndex-this.limit) && p.id>(this.pastIndex-this.limit))
         --this.page
        }
      }else {
        this.sortBooks();
      }
    },
    changeIntoTable(){
       this.collections[this.selectedRow-1].author = this.inputAuthor;
       this.collections[this.selectedRow-1].name = this.inputName;
       this.collections[this.selectedRow-1].closet = this.inputCloset;
       this.collections[this.selectedRow-1].year = this.inputYear;
       this.collections[this.selectedRow-1].themen = this.inputThemen;
       this.collections[this.selectedRow-1].category = this.inputCategory;
       this.inputAuthor = "";
       this.inputName = "";
       this.inputCloset = "";
       this.inputYear = "";
       this.inputThemen = "";
       this.inputCategory = "";
    },
    isNumeric(n) {
      return !isNaN(parseFloat(n)) && isFinite(n);
    },
    sortBooks() {
      if (this.sortOption == 0){
        this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex)
      }else{
        let sortedArray = this.collections.slice(0)
        if (this.isNumeric(this.sortOption))
          sortedArray.sort((book1, book2) => book1[this.sortOption]-book2[this.sortOption]);
        else
          sortedArray.sort((book1, book2) => book1[this.sortOption].localeCompare(book2[this.sortOption]));
        if (this.pastIndex>0)
          this.collectionsPage = sortedArray.slice(this.pastIndex-1, this.currentIndex);
        else
          this.collectionsPage = sortedArray.slice(this.pastIndex, this.currentIndex);
      }
    },
    search(){
      if (this.searchOption != -1){
        let searchArr
        if(this.searchOption instanceof String)
          searchArr = this.collections.filter(book => book[this.searchOption].includes(this.searchQuery));
        else
        searchArr = this.collections.filter(book => book[this.searchOption].toString().includes(this.searchQuery));
        this.totalPages = Math.ceil(searchArr.length / this.limit);
        if (this.pastIndex>0)
          this.collectionsPage = searchArr.slice(this.pastIndex-1, this.currentIndex);
        else
          this.collectionsPage = searchArr.slice(this.pastIndex, this.currentIndex);
        }
    },
    exportToExcel(type, fn, dl){
      let l = this.limit;
      this.limit = this.collections.length;
      let elt = document.getElementById('tbl_exporttable_to_xls');
      let wb = XLSX.utils.table_to_book(elt, { sheet: "sheet1" });
      this.limit = l;
      return dl ?
        XLSX.write(wb, { bookType: type, bookSST: true, type: 'base64' }):
        XLSX.writeFile(wb, fn || ('BooksTable.' + (type || 'xlsx')));  
    }
  },
  watch: {
      limit(){
        this.totalPages = Math.ceil(this.collections.length / this.limit);
        this.currentIndex = this.limit;
        this.collectionsPage = this.collections.filter(p => p.id<=this.currentIndex && p.id>this.pastIndex);
      }
  },
  
};
</script>

<style>
.logo{
  height: 80%;
  width: 60%;
  margin-top: 15px;
  margin-left: 15px;
}
.page__wrapper{
  display: flex;
  margin-top: 15px;
  margin-bottom: 15px;
  margin-left: 10px;
  margin-right: auto;
}
.page{
  border: 1px solid black;
  padding: 10px;
  margin-left: 5px;
}
.current-page{
  border: 2px solid teal;
}
.tableSec{
  display: block;
  margin-left: auto;
  margin-right: auto;
  width: 98%;
}
.waitLabel{
  margin-left: auto;
  margin-right: auto;
  text-align: center;
  width: 95%;
  font-size: 25px;
  color: red;
}
.inputFile{
  margin-right: auto;
  margin-top: 48px;
  width: 75%;
  height: 75%;
}
.activeItem{
  border: 10px solid #4285B4;
  background-color: #4285B4;
}
.form{
  display: flex;
  flex-direction: column;
  margin: 20px;
}
.input{
  width: 100%;
  border: 1px solid #4285B4;
  padding: 10px 15px;
  margin-top: 15px;
  margin-left: auto;
  margin-right: auto;
}
._btn{
  margin-top: 15px;
  align-self: flex-end;
  padding: 10px 15px;
  background-color: none;
  color: #4285B4;
  border: 1px solid #4285B4;
}
.btn_in_div{
  padding: 10px 15px;
  background-color: none;
  color: #4285B4;
  border: 1px solid #4285B4;
}
</style>