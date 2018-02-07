<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <input type="file" name="file" @change="processFile($event)"/>
    <ul>
      <li>
        <a
          href="http://chapilabs.com"
          target="_blank"
        >
          Chapi Labs &copy; {{ year }}
        </a>
      </li>
    </ul>
  </div>
</template>

<script>
import XLSX from 'xlsx';
var DBFFile = require('dbffile');
const save = require('save-file')
import dbf from 'dbf';
import moment from 'moment';

export default {
  name: 'Main',
  data () {
    return {
      msg: 'Convertidor XLSX a DBF (dbase III)',
      year: new Date().getFullYear()
    }
  },
  methods: {
    toBuffer(ab) {
      var buffer = new Buffer(ab.byteLength);
      var view = new Uint8Array(ab);
      for (var i = 0; i < buffer.length; ++i) {
          buffer[i] = view[i];
      }
      return buffer;
    },
    getType(cellType) {
      if (cellType === undefined) return 'C';
      const type = cellType.t;
      switch (type) {
        case 'n':
          return 'N';
        case 'd':
          return 'D';
        case 'b':
          return 'L';
        case 's':
          return 'C';
        default:
          return 'C';
      }
    },
    processFile(e) {
      let _this = this;
      var rABS = true;
      var files = e.target.files, f = files[0];
      var reader = new FileReader();
      let counter = 0;
      const format = 'YYYY-DD-MM h mm a';

      reader.onload = function(e) {
        let data = e.target.result;
        if(!rABS) data = new Uint8Array(data);
        const workbook = XLSX.read(data, {type: rABS ? 'binary' : 'array'});
        const sheet_name_list = workbook.SheetNames;
        const json_data = (XLSX.utils.sheet_to_json(workbook.Sheets[sheet_name_list[0]]));
        //console.log(json_data);
        // const sheet = workbook.Sheets[sheet_name_list[0]];
        // var range = XLSX.utils.decode_range(sheet['!ref']); // get the range
        // let fieldDescriptors = [];
        // let rows = [];
        // let header = 1;
        // for(var R = range.s.r; R <= range.e.r; ++R) {
        //   let row = {};
        //   for(var C = range.s.c; C <= range.e.c; ++C) {

        //     /* find the cell object */
        //     var cellref = XLSX.utils.encode_cell({c:C, r:R}); // construct A1 reference for cell
        //     //if(!sheet[cellref]) continue; // if cell doesn't exist, move on
        //     var cell = sheet[cellref];

        //     if (header === 1) {
        //       fieldDescriptors.push({
        //         name: cell.v,
        //         type: 'C',
        //         size: 20
        //       })
        //     }
        //     if (header === 2) {
        //       fieldDescriptors[counter].type = _this.getType(cell)
        //     }
        //     if (header !== 1) {
        //       row[fieldDescriptors[counter].name] = cell === undefined ? '': cell.v;

        //     }
        //     counter = counter + 1;
        //   }
        //   if (header !== 1) {
        //     rows.push(row);
        //   }

        //   header += 1;
        //   counter = 0;
        // }
        // console.log(fieldDescriptors);
        // console.log(rows);
        // console.log(DBFFile);
        let fixed_array = [];
        for (let i = 0; i < json_data.length; i++) {
          let keys = Object.keys(json_data[i]);
          let createObj = {};

          for (let j = 0; j < keys.length; j++) {
            createObj[keys[j]] = json_data[i][keys[j]];
          }
          fixed_array.push(createObj);
        }
        console.log(fixed_array);
        var buf = dbf.structure(fixed_array);
        const buffer = _this.toBuffer(buf.buffer);
        save(buffer, 'data.dbf', (err, data) => {
          if (err) throw err;

            //file is saved at this point, data is arrayBuffer with actual saved data
        });


        /* DO SOMETHING WITH workbook HERE */
      };
      if(rABS) reader.readAsBinaryString(f); else reader.readAsArrayBuffer(f)

    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
