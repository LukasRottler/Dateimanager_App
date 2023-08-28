<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Dateimanager</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <!-- Button zum Hinzufügen eines Ordners -->
      <ion-button @click="addFolder">Add Folder</ion-button>
      <!-- Button zum Auswählen einer Datei -->
      <ion-button @click="pickFile">Pick File</ion-button>
      <!-- Button zum Laden von Ordnern -->
      <ion-button @click="loadFolders">Load Folders</ion-button>
      <!-- Button zum Zurückgehen -->
      <ion-button v-if="path !== '/'" @click="goBack">Go Back</ion-button>
      <!-- Liste der Ordner -->
      <ion-list>
        <ion-item>
          Ordner: 
        </ion-item>
        <ion-item v-for="(folder, index) in folders" :key="folder.name">
          <ion-label @click="openFolder(folder.name)">{{ folder.name }}</ion-label>
          <!-- Button zum Löschen eines Ordners -->
          <ion-button @click="deleteFolder(index)">Delete</ion-button>
        </ion-item>
        <ion-item>
          Dateien: 
        </ion-item>
      </ion-list>
      <!-- Liste der im LIBRARY-Verzeichnis befindlichen Dateien -->
      <ion-list>
        <ion-item v-for="(file, index) in libraryFiles" :key="file.name">
      <ion-label @click="openFile(file.name)">{{ file.name }}</ion-label>
      <!-- Button zum Löschen einer Datei -->
      <ion-button @click="deleteFile(index)">Delete</ion-button>
    </ion-item>
      </ion-list>
    </ion-content>
  </ion-page>
</template>
<script>
import { Plugins } from '@capacitor/core';
import { IonButton, IonHeader, IonList, IonToolbar,IonContent,IonPage,IonTitle,IonLabel, IonItem } from '@ionic/vue';
import { FilePicker } from '@capawesome/capacitor-file-picker';
import { FileOpener } from '@capawesome-team/capacitor-file-opener'; // Import des File-Opener Plugins
const {  Filesystem, Directory,Encoding  } = Plugins;


export default {
  components: {
    // Hier werden die importierten Ionic-Vue-Komponenten registriert
    IonButton,
    IonHeader,
    IonList,
    IonToolbar,
    IonContent,
    IonPage,
    IonTitle,
    IonLabel,
    IonItem
},
  data() {
    return {
      folders: [], // Liste der Ordner
    
      path: '/',   // Pfad für die readdir Methode
      directory: "LIBRARY", // Verzeichnis für die readdir Methode
      libraryFiles: [] // Liste der im LIBRARY-Verzeichnis befindlichen Dateien
    };
  },
  
  methods: {
    async addFolder() {
      const folderName = prompt('Enter folder name:');
      if (folderName) {
        try {
          // Erstelle den Ordner mit Filesystem API im LIBRARY-Verzeichnis
          await Filesystem.mkdir({
            path: `${this.path}/${folderName}`,
            directory: "LIBRARY",
            recursive: false
          });
          console.log('Folder created successfully.');
          // Füge den erstellten Ordner zur Liste der Ordner hinzu
          this.folders.push({ name: folderName });
        } catch (error) {
          console.error('Error creating folder:', error);
        }
      }
    },
    async goBack() {
      // Extrahiere den vorherigen Ordnerpfad
      const parts = this.path.split('/').filter(part => part !== '');
      parts.pop();
      this.path = '/' + parts.join('/');

      // Lade die Dateien des vorherigen Ordners
      await this.loadFolders();
    },
    async openFolder(folderName) {
      try {
        console.log('OpenFolder')
        // Verändere den Pfad zum geöffneten Ordner
        this.path = `/${folderName}`;
        const { files } = await Filesystem.readdir({
          path: this.path,
          directory: this.directory
        });

        // Füge alle gefundenen Einträge zur Liste der Ordner hinzu
          this.folders = files.map(entry => ({ name: entry.name }));
       
          const filePath = `${this.directory}/${folderName}`;
        
      } catch (error) {
        console.error('Error opening folderr:', error);
      }
    },
    async openFile(fileName) {
      try {
        const  {uri}  = await Filesystem.getUri({
        path: fileName,
        directory: 'LIBRARY',
      });
      this.pathb = uri;

    console.log('Uri:',uri)
    console.log('This Path', this.pathb)

      await FileOpener.openFile({ path: this.pathb});
      console.log('File opened');
        console.log(`File "${fileName}" opened successfully.`);
      } catch (error) {
        console.error('Error opening file:', error);
      }
    },
    async pickFile() {
    console.log('Filepicker aktiv');
    try {
      const pickedFiles = await FilePicker.pickFiles();
      if (Array.isArray(pickedFiles)) {
        console.log("It's an array");
      } else if (typeof pickedFiles === "object") {
        console.log("It's a generic object");
      } else if (typeof pickedFiles === "function") {
        console.log("It's a function");
      } else {
        console.log("Unknown type");
      }
      const pickedFile = pickedFiles.files[0]
      
      console.log('Picked files:', pickedFiles);
      console.log('File URI:', pickedFile.path);
      console.log('File Name:', pickedFile.name);
      console.log('File Type:', pickedFile.type);
    
        console.log('Picked file:', pickedFile);
        //console.log('File URI:', pickedFile.uri);
        console.log('File Name:', pickedFile.name);
        console.log('File Type:', pickedFile.type);

        if (pickedFile && pickedFile.path) {
          console.log('File URI:', pickedFile.path);
          console.log('File Name:', pickedFile.name);
          console.log('File Type:', pickedFile.type);

          const fileContent = await this.readFileContent(pickedFile.path);
          console.log('Content',fileContent)
          await this.writeFileToLibrary(pickedFile.name, fileContent);

          // Füge die ausgewählte Datei zur Liste der Dateien hinzu
          this.files.push({ name: pickedFile.name });

          console.log(`File "${pickedFile.name}" added to LIBRARY directory.`);
        
      }
    } catch (error) {
      console.error('Error picking files:', error);
    }
  },
    

    async readFileContent(uri) {
      try {
        const { data } = await Filesystem.readFile({ path: uri });
        console.log('ReadFileContent erfolgreich')
        return data;
      } catch (error) {
        console.error('Error reading file content:', error);
        throw error;
      }
    },

    async writeFileToLibrary(fileName, fileContent) {
      try {
        await Filesystem.writeFile({
          path: `/${fileName}`,
          data: fileContent,
          directory: "LIBRARY",
        });
      } catch (error) {
        console.error('Error writing file to LIBRARY directory:', error);
        throw error;
      }
    },

    // ... (weitere Methdoden, unverändert) ...
  
    async loadFolders() {
      try {
        const { publicStorage } = await Filesystem.checkPermissions();
        if (publicStorage !== 'granted') {
          await Filesystem.requestPermissions();
        }

        const { files } = await Filesystem.readdir({
          path: this.path,
          directory: this.directory
        });

         // Teile die Dateien in Ordner und Dateien auf
         const folders = files.filter(entry => entry.type === 'directory');
        const filesOnly = files.filter(entry => entry.type !== 'directory');

        // Füge die Ordner zur Liste der Ordner hinzu
        this.folders = folders.map(entry => ({ name: entry.name }));

        // Füge die Dateien zur Liste der Dateien hinzu
        this.libraryFiles = filesOnly.map(entry => ({ name: entry.name }));

      } catch (error) {
        console.error('Error loading folders:', error);
      }
    },
    
  
    async deleteFolder(index) {
      try {
        const folderName = this.folders[index].name;
        if (confirm(`Are you sure you want to delete the folder "${folderName}"?`)) {
          // Lösche den Ordner aus der Liste
          this.folders.splice(index, 1);

          // Lösche den Ordner im LIBRARY-Verzeichnis
          await Filesystem.rmdir({
            path: `/${folderName}`,
            directory: "LIBRARY",
            recursive: true
          });

          console.log('Folder deleted successfully.');
        }
      } catch (error) {
        console.error('Error deleting folder:', error);
      }
    },
    async deleteFile(index) {
      try {
        const fileName = this.libraryFiles[index].name;
        if (confirm(`Are you sure you want to delete the file "${fileName}"?`)) {
          // Lösche die Datei aus der Liste
          this.libraryFiles.splice(index, 1);

          // Lösche die Datei im LIBRARY-Verzeichnis
          await Filesystem.deleteFile({
            path: fileName,
            directory: "LIBRARY",
          });

          console.log('File deleted successfully.');
        }
      } catch (error) {
        console.error('Error deleting file:', error);
      }
    },

  }
};

</script>