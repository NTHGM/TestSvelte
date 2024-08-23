<script lang="ts">
    import { XMLParser } from "fast-xml-parser";
    import { XMLBuilder } from "fast-xml-parser";
    import Handsontable from "handsontable";
    import "handsontable/dist/handsontable.full.min.css";

    let k = new Map<String, Array<String>>([]);
    let str = new Array();
    let strT;
    let strSettings = {
        colHeaders: ["Key", "Default"],
        rowHeaders: true,
        height: "auto",
        autoWrapRow: true,
        autoWrapCol: true,
        with: "auto",
        minSpareRows: 1,
    };

    let first = true;

    function loadTable(datas, id, table) {
        if (table) {
            loadData(table, datas);
            return table;
        }
        let container = document.getElementById(id);
        let setting = strSettings;

        let hot = new Handsontable(container, {
            data: datas,
            ...setting,
            licenseKey: "non-commercial-and-evaluation", // for non-commercial use only
        });
        //neu map lai thi can tham chieu den jsonObj.resources
        return hot;
    }

    function loadData(hot, data) {
        hot.loadData(data);
    }

    function OnUploadFile(event) {
        const file = event.target.files[0];
        if (file) {
            XMLtoJSON(file);
        }
        event.target.value = null;
    }

    //parser-builder
    const options = {
        ignoreAttributes: false,
        attributeNamePrefix: "@_",
        allowBooleanAttributes: true,
        suppressBooleanAttributes: false,
        format: true,
    };
    const parser = new XMLParser(options);

    function XMLtoJSON(file) {
        const reader = new FileReader();
        reader.onload = function (e) {
            const xmlContent = e.target.result;
            let jsonObj = parser.parse(xmlContent);
            SetupVar(jsonObj);
        };
        reader.readAsText(file);
    }

    function SetupVar(jsonObj) {
        let strL = new Array();
        if (jsonObj.resources["string"].length < 2) {
            strL = [...strL, jsonObj.resources["string"]];
            strL = strL.map((item) => ({
                name: item["@_name"],
                text: item["#text"],
            }));
        } else
            strL = jsonObj.resources["string"].map((item) => ({
                name: item["@_name"],
                text: item["#text"],
            }));

        if (first) {
            strL.forEach((e) => {
                k.set(e.name, [e.text]);
            });
            first = false;
        } else {
            let length = k.values().next().value.length;
            let h = new Map();
            for (let x of k.keys()) {
                h.set(x, null);
            }
            strL.forEach((e) => {
                h.delete(e.name);
                if (k.has(e.name)) {
                    k.get(e.name).push(e.text);
                } else {
                    let a = Array();
                    for (let i = 0; i < length; i++) {
                        a.push(" ");
                    }
                    a = [...a, e.text];
                    k.set(e.name, a);
                }
            });
            for (let i of h.keys()) {
                let x = k.get(i);
                x.push(" ");
            }
            console.log(k);
        }
        str.splice(0, str.length);
        for (let i of k.keys()) {
            let x = k.get(i);
            let obj = {};
            obj["0"] = i;
            for (let i = 0; i < x.length; i++) {
                obj["" + (i + 1)] = x[i];
            }
            str = [...str, obj];
        }
        console.log(str);
        strT = loadTable(str, "stringT", strT);
    }
    const builder = new XMLBuilder(options);
    let ori = null;
    function JSONtoXML() {
        //ori = builder.build(jsonObj);
    }

    function openTab(evt, type) {
        let i, tabcontent;
        tabcontent = document.getElementsByClassName("tabcontent");
        for (i = 0; i < tabcontent.length; i++) {
            tabcontent[i].style.display = "none";
        }
        document.getElementById(type).style.display = "block";
    }
</script>

<div class="sidenav">
    <div
        style="height:70px; text-align:center; 
	background-color:gray; padding:25px 0px;"
    >
        <input
            on:change={OnUploadFile}
            id="upload"
            type="file"
            style="display:none;"
        />
        <label for="upload" class="but">Upload</label>
    </div>
    <div>
        <button on:click={(event) => openTab(event, "string")} id="defaultOpen"
            >String</button
        >
        <button on:click={(event) => openTab(event, "plural")}>Plural</button>
        <button on:click={(event) => openTab(event, "stringarray")}
            >String-Array</button
        >
    </div>
</div>

<div class="content">
    <div id="string" class="tabcontent">
        <h2>String</h2>
        <div id="stringT"></div>
    </div>

    <div id="plural" class="tabcontent">
        <h2>Plural</h2>
        <div id="pluralT"></div>
    </div>

    <div id="stringarray" class="tabcontent">
        <h2>String-Array</h2>
        <div id="stringarrayT"></div>
    </div>
</div>

<style>
    * {
        box-sizing: border-box;
    }

    .sidenav {
        height: 100%;
        width: 200px;
        position: fixed;
        z-index: 1;
        top: 0;
        left: 0;
        background-color: #111;
        overflow-x: hidden;
    }

    .sidenav button {
        color: white;
        padding: 16px;
        text-decoration: none;
        display: block;
        width: 100%;
    }

    .sidenav button:hover {
        background-color: #ddd;
        color: black;
    }
    .sidenav button:active {
        background-color: #924040;
        color: black;
    }
    .content {
        margin-left: 200px;
        padding-left: 20px;
    }

    .but {
        border: 2px solid black;
        border-radius: 5px;
        text-align: center;
        width: 40%;
        background-color: white;
        padding: 2px;
    }

    .but:hover {
        background-color: black;
        color: white;
        border: 2px solid white;
    }
    .but:active {
        background-color: gray;
        color: white;
        border: 2px solid white;
    }

    .tabcontent {
        display: none;
    }
</style>
