<template>
  <div id="app">
    <ws_grid
      :ws_data="ws_data"
      @remove_all="remove_all"
      @ws_send="ws_send"
      @export_all="export_all"
      @import_ws="import_ws"
    />
  </div>
</template>

<script>
	import ws_grid from './components/ws_grid/ws_grid.vue';

	export default {
		name: 'Home',
		components: { ws_grid: ws_grid	},
		data() {
			return {
				ws_data: []
			};
		},
		methods: {
			remove_all() {
				this.ws_data = [];
			},
			ws_send(item_data) {
				let d;
				try {
					d = JSON.stringify(JSON.parse(item_data));
				} catch (e) {
					d = item_data.toString();
				}

				this.ws_data.push({
					type: 'to',
					data: d,
					length: item_data.length,
					from_devtools: true,
					time: new Date()
				});
			},
      import_ws() {
			  //alert('import works');
        let import_dialog = document.createElement('input');
        import_dialog.type = 'file';
        let ws_data = this.ws_data;
        import_dialog.onchange = e => {
          let file = e.target.files[0];
          let reader = new FileReader();
          reader.readAsText(file,'UTF-8');
          reader.onload = readerEvent => {
            let content = readerEvent.target.result; // this is the content!
            content.split('\n').forEach(function(csv_string,index){
              if ( csv_string.startsWith('sep=') ) {
                return;
              }
              if ( csv_string.startsWith('type|') ) {
                return;
              }
              if (csv_string.length < 5) {
                return;
              }
              let tmp_data = {};
              csv_string.split('|').forEach(function(csv_item,index){
                switch (index) {
                  case 0:  // type
                    tmp_data.type = csv_item;
                    break;
                  case 1:  // data
                    //tmp_data.data = JSON.parse(csv_item);
                    tmp_data.data = csv_item;
                    break;
                  case 2:  // length
                    tmp_data.length = csv_item;
                    break;
                  case 3:  // time
                    tmp_data.time = new Date();
                    tmp_data.time.setTime(csv_item);
                    break;
                  case 4:  // formatted_data
                    //tmp_data.formatted_data = csv_item;
                    break;
                  case 5:  // formatted_time
                    //tmp_data.formatted_time = csv_item;
                    break;
                  case 6:  // class
                    tmp_data.from_devtools = false;
                    //tmp_data.class = csv_item;
                    break;
                  default:
                    break;
                }
              });
              if ( index > 51 ) {
                // alert(JSON.stringify(tmp_data));
              }
              ws_data.push(tmp_data);
            });
          }
        }
        document.body.appendChild(import_dialog);
        import_dialog.click();
        document.body.removeChild(import_dialog);
      },
      export_all() {
        // Building the CSV from the Data two-dimensional array
        // Each column is separated by ";" and new line "\n" for next row
        let csvContent = 'sep=|\n';
        csvContent += 'type|data|length|time|formatted_data|formatted_time|class\n';
        let dataString = '';
        this.ws_data.forEach(function(item) {
          let date = new Date(item.time);
          dataString = '' + item.type + '|' + item.data + '|' + item.length + '|' + date.getTime() + '|' + item.formatted_data + '|' + item.formatted_time + '|' + item.class + '\n';
          csvContent += dataString;
        });

        // The download function takes a CSV string, the filename and mimeType as parameters
        // Scroll/look down at the bottom of this snippet to see how download is called
        let download = function(content, fileName, mimeType) {
          let a = document.createElement('a');
          mimeType = mimeType || 'application/octet-stream';

          if (navigator.msSaveBlob) { // IE10
            navigator.msSaveBlob(new Blob([content], {
              type: mimeType
            }), fileName);
          }
          else if (URL && 'download' in a) { //html5 A[download]
            a.href = URL.createObjectURL(new Blob([content], {
              type: mimeType
            }));
            a.setAttribute('download', fileName);
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
          }
          else {
            location.href = 'data:application/octet-stream,' + encodeURIComponent(content); // only this mime type is supported
          }
        }

        download(csvContent, 'dowload.csv', 'text/csv;encoding:utf-8');
			}
		},
	};
</script>

<style>
    body {
        padding: 0;
        margin: 0;
        font-family: 'helvetica neue', helvetica, arial, 'lucida grande', sans-serif;
        font-size: 12px;
    }

    #app {
        display: flex;
        flex-flow: row wrap;
        /*align-items: center;*/
        align-content: center;
        justify-content: space-between;
    }

    .grid {
        width: inherit;
        display: flex
    }

    .frame {
        width: 35%;
        display: flex;
        align-items: baseline;
        margin-top: 10px;
        position: fixed;
        right: 0px;
        top: 40px;
        height: 100%;
    }
</style>
