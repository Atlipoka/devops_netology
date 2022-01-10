1. Поиск производил коммандой git show aefea. Получил полную информацию по комиту, хэш комита - commit aefead2207ef7e2aa5dc81a34aedf0cad4c32545, а комментарий - Update CHANGELOG.md
2. Поиск производил коммандой git show 85024d3. Данному хэшу был присвое тэг - tag: v0.12.23.
3. Поиск производил коммандой git show 85024d3 и для поиска родителей коммита использовал комманду git show b8d720^. У данного коммита 2 родителя, их хэши commit 56cd7859e05c36c06b56d013b55a252d0bb7e158 и commit 9ea88f22fc6269854151c571162c5bcf958bee2b.
4. Поиск производил коммандой  git log v0.12.23..v0.12.24 --oneline --graph. Список всех коммитом и комментариев:
 * b14b74c49 [Website] vmc provider links
 * 3f235065b Update CHANGELOG.md
 * 6ae64e247 registry: Fix panic when server is unreachable
 * 5c619ca1b website: Remove links to the getting started guide's old location
 * 06275647e Update CHANGELOG.md
 * d5f9411f5 command: Fix bug when using terraform login on Windows
 * 4b6d06cc5 Update CHANGELOG.md
 * dd01a3507 Update CHANGELOG.md
 * 225466bc3 Cleanup after v0.12.23 release
5. Поиск файла, в которм создавалась функция providerSource, производил по комманде git grep -n 'func providerSource(', далее используя комманду git log -L :providerSource:provider_source.go нашел все коммиты, где фигурирует название данной функции и нашел коммит commit 8c928e83589d90a031f811fae52a81be7153e82f, в которм была создана данная функция.
6. Поиск файла, в которм фигурирует функция globalPluginDirs искал по команде  git grep -n 'func globalPluginDirs(', далее искад все комиты по команде git log -L :globalPluginDirs:plugins.go, список коммитов:
 * 8364383c359a6b738a436d1b7745ccdce178df47
 * 66ebff90cdfaa6938f26f908c7ebad8d547fea17
 * 41ab0aef7a0fe030e84018973a64135b11abcd70
 * 52dbf94834cb970b510f2fba853a5b49ad9b1a46
 * 78b12205587fe839f10d946ea3fdc06719decb05
7. При помощи команды git log -S 'func synchronizedWriters' нашел коммит, в котром была создана функция synchronizedWriters, автор функции - Martin Atkins <mart@degeneration.co.uk>.
