# AutoSaveManager for Unity

**Author:** atiliodev ([GitHub](https://github.com/atiliodev))

> [Read in English](#english-version) | [Leia em Português](#versão-em-português)

---

<a id="english-version"></a>
## 🇬🇧 English Version

Welcome to the AutoSaveManager documentation for Unity. This document covers everything from installation and basic usage to the deep code architecture for developers who wish to understand it.

### 1. Overview
AutoSaveManager is an Editor tool for Unity focused on preventing progress loss during development. It acts silently and performantly in the background of the engine, tracking key events and saving the project based on configurable rules and constraints.
**Attention:** Because it is a tool contained within the `Editor` folder, no code or weight from this asset will be compiled into the final build of your game.

### 2. Installation & Access
1. **Import:** Import the `.unitypackage` or place the `AutoSaveManager` folder directly into your project's `Assets` directory.
2. **Required Path:** Make sure the scripts remain inside a folder named `Editor` (e.g., `Assets/AutoSaveManager/Editor/`) so Unity knows they belong exclusively to the Editor.
3. **Opening the Window:** On Unity's standard top bar, navigate to **Window > Auto Save Manager**.
   *It is recommended to dock the tab somewhere close to the Inspector or at the bottom near the Console.*

### 3. Configuration Guide (The Interface)
The interface is divided into logical blocks. The panel reacts to your modifications immediately, saving your preferences using `EditorPrefs` (your preferences remain local to your PC and will never cause Git/Version Control conflicts with your team).

#### Triggers & Conditions
* **Save Interval (Minutes):** A classic timer. Defines how often Unity will automatically save (Limits from 1 to 60 minutes).
* **Save before entering Play Mode:** When checked, it forces a save of your Scenes before compiling scripts and starting the play mode. This prevents crashes upon Play Mode initialization.
* **Save when Editing (Undo/Redo):** Listens to Unity's "undo" buffer. If you move many objects or make massive property changes that Unity can track (Undo dirty), it triggers a save.
* **Save on Hierarchy Changes:** Triggers the saving routine if you create, duplicate, or delete GameObjects in the Hierarchy window.
* **Save on Inactivity:** A safety net. If you take your hands off the keyboard and move the mouse away from the Scene window for the specified **Inactivity Limit**, the Asset assumes you are idle and performs a silent save.

#### Advanced
* **Show Console Logs:** If checked, it will place a non-intrusive message (green and stylized) in the Unity Console whenever an auto-save happens successfully.

### 4. Code Architecture (For Developers)
The Asset is contained entirely within the `AutoSaveManager.Editor` namespace. It was designed under the Data-Logic-View model (Data, Internal logic, and UI).

#### A. AutoSaveSettings.cs
A simple yet effective static repository.
* Instead of generating `ScriptableObjects` (which would dirty your team's commits if multiple people install it with different configs), it relies on accessor functions (get/set) that interact with Unity's native `EditorPrefs` class.
* The strict prefix `AutoSaveManager_` on all keys ensures it doesn't overwrite local keys created by other extensions.

#### B. AutoSaveLogic.cs
The pulsing engine of the asset.
* **[InitializeOnLoad]:** This tag above the static declaration ensures Unity invokes the class constructor as soon as the engine finishes loading. In other words, it is running in memory right from the start.
* **Associated Events:**
  * `EditorApplication.update`: Replaces the common `Update()` in Monobehaviours to count the `TimeSinceStartup` inside the editor ecosystem.
  * `Undo.undoRedoPerformed`: Triggered after modified data hits the engine's action-reverting container.
  * `EditorApplication.playModeStateChanged`: Specifically checks the `ExitingEditMode` enumerator.
  * `SceneView.duringSceneGui`: Injects an invisible listener over the Scene's rendered viewport. This is vital to check if an `Event.current` exists (meaning your mouse and keyboard are moving in the preview = active user).
* **Anti-fraud Security:** The `IsAnySceneDirty()` function iterates through the `EditorSceneManager.sceneCount` array ensuring we don't intrude on processes by saving scenes that don't need to be saved. Furthermore, the filtering blocks scenarios where the scene is "Untitled" (`string.IsNullOrEmpty(scene.path)`).

#### C. AutoSaveWindow.cs
The IMGUI user interface.
* Uses pre-saved `GUIStyle` blocks under a standalone `SetupStyles()` method to generate zero allocations in memory during `OnGUI()`.
* Utilizes heavy RichText from the HTML/HEX color palette to look modern alongside classic and robust code.

> **Customization Tip:** If you need to force the AutoSave to also save the project state in PlasticSCM/Git or close other files on disk, go to the `AutoSaveLogic` class at the end of the `PerformSave()` method (right after the `AssetDatabase.SaveAssets();` line) and add your custom console batch calls for your third-party tool!

---

<br>

<a id="versão-em-português"></a>
## 🇧🇷 Versão em Português

Bem-vindo à documentação do AutoSaveManager para Unity. Este documento cobre desde a instalação e uso básico até a arquitetura profunda do código para desenvolvedores que desejam entende-lo.

### 1. Visão Geral
O AutoSaveManager é uma ferramenta de Editor para a Unity focada em prevenir a perda de progresso durante o desenvolvimento. Ele atua de forma silenciosa e performática no background da engine, rastreando eventos chaves e salvando o projeto baseado em restrições e regras configuráveis.
**Atenção:** Como é uma ferramenta contida na pasta `Editor`, nenhum código ou peso deste asset será compilado no build final do seu jogo.

### 2. Instalação & Acesso
1. **Importação:** Importe o pacote `.unitypackage` ou coloque a pasta `AutoSaveManager` diretamente no diretório `Assets` do seu projeto.
2. **Caminho Obrigatório:** Certifique-se de que os scripts permaneçam dentro de uma das pastas chamadas `Editor` (ex: `Assets/AutoSaveManager/Editor/`) para que a Unity saiba que eles pertencem apenas ao Editor.
3. **Abertura da Janela:** Na barra superior padrão da Unity, vá até **Window > Auto Save Manager**.
   *O recomendável é ancorar (dock) a aba em algum lugar próximo ao Inspector ou no rodapé próximo ao Console.*

### 3. Guia de Configurações (A Interface)
A interface é dividida em blocos lógicos. O painel reage às suas modificações de maneira imediata, salvando suas preferências usando `EditorPrefs` (suas preferências ficam locais no seu PC e nunca causam conflitos de Git/Versionamento com sua equipe).

#### Triggers & Conditions (Gatilhos)
* **Save Interval (Minutes):** Um temporizador clássico. Define de quanto em quanto tempo a Unity será salva automaticamente (Limites de 1 a 60 minutos).
* **Save before entering Play Mode:** Ao estar marcado, ele força o salvamento das suas Cenas antes de compilar os scripts e iniciar o modo de execução. Previne travamentos na inicialização do Play Mode.
* **Save when Editing (Undo/Redo):** Escuta o buffer de "desfazer" da Unity. Se você moveu muitos objetos ou mudou propriedades massivas que a Unity consiga rastrear (Undo dirty), ele engatilha o save.
* **Save on Hierarchy Changes:** Dispara a rotina de salvamento caso você crie, duplique ou exclua GameObjects na janela de Hierarquia.
* **Save on Inactivity:** Uma rede de segurança. Se você afastar as mãos do teclado e tirar o mouse da janela de Cena pelo tempo estipulado em **Inactivity Limit**, o Asset entende que você está ocioso e faz o salvamento silencioso.

#### Advanced
* **Show Console Logs:** Se marcado, irá colocar uma mensagem não-intrusiva (cor verde e estilizada) no Unity Console sempre que um auto-save acontecer com sucesso.

### 4. Arquitetura de Código (Para Desenvolvedores)
O Asset é contido inteiramente no namespace `AutoSaveManager.Editor`. Foi desenhado no modelo Data-Logic-View (Dados, Lógica interna e UI).

#### A. AutoSaveSettings.cs
Um repositório estático simples mas efetivo.
* Em vez de gerar `ScriptableObjects` (que sujariam os commits da sua equipe caso várias pessoas instalem com configs diferentes), ele engloba funções acessoras (get/set) que interagem com a classe nativa da Unity `EditorPrefs`.
* O prefixo rígido `AutoSaveManager_` em todas as keys garante que ele não reescreva chaves locais acriticamente criadas por outras extensões.

#### B. AutoSaveLogic.cs
O motor pulsante do ativo.
* **[InitializeOnLoad]:** Essa tag acima da declaração estática garante que a Unity invoque o construtor da classe assim que a engine termina de carregar. Ou seja, ela já nasce executando na memória.
* **Eventos Associados:**
  * `EditorApplication.update`: Substitui o `Update()` comum em Monobehaviours para poder contar o `TimeSinceStartup` dentro do ecossistema do editor.
  * `Undo.undoRedoPerformed`: Ativado depois que dados modificados chegam no container de reverter ações da engine.
  * `EditorApplication.playModeStateChanged`: Checa especificamente o enumerador `ExitingEditMode`.
  * `SceneView.duringSceneGui`: Injeta um listener invisível sobre a viewport renderizada das Cenas. Isso é vital para checar se uma verificação `Event.current` existe (ou seja, seu mouse e teclado estão movimentando o preview = usuário ativo).
* **Segurança Antifraude:** A função `IsAnySceneDirty()` percorre o array `EditorSceneManager.sceneCount` assegurando de que não invadimos processos gravando cenas que não precisam ser salvas. Além disso, a filtragem barra cenários onde a cena é "Untitled" (`string.IsNullOrEmpty(scene.path)`).

#### C. AutoSaveWindow.cs
A interface do usuário IMGUI.
* Usa `GUIStyle` em bloco pre-salvo sob o método isolado `SetupStyles()` para gerar zero alocações na memória durante o `OnGUI()`.
* Utiliza RichText pesado da paleta de cores HTML/HEX para se parecer moderno com código clássico e robusto.

> **Dica de Customização:** Se você precisa forçar que o AutoSave também salve o estado do projeto no PlasticSCM/Git ou feche outros arquivos em disco, vá até a class `AutoSaveLogic` no fim do método `PerformSave()` (logo após a linha `AssetDatabase.SaveAssets();`) e adicione as chamadas em batch do console para sua ferramenta terceira!
