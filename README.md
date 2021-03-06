ClipperHTML
```
const template = `
<script>
    var iframe = document.getElementById('r-iframe');
    function pageY(elem) {
        return elem.offsetParent ? (elem.offsetTop + pageY(elem.offsetParent)) : elem.offsetTop;
    }
    function resizeIframe() {
        var height = document.documentElement.clientHeight;
        height -= pageY(iframe) + 20 ;
        height = (height < 0) ? 0 : height;
        iframe.style.height = height + 'px';
    }
    iframe.onload = resizeIframe;
    window.onresize = resizeIframe;
</script>
<iframe id="r-iframe" style="width:100%" src="data:text/html;charset=utf-8,%%CONTENT%%" sandbox=""></iframe>
`;

const {req, res} = api;
const {title, url, content} = req.body;

if (req.method == 'POST') {
    api.log("==========================");

    //const todayNote = await api.getDateNote(today);
    const todayNote = await api.getTodayNote();
    
    // create render note
    const renderNote = (await api.createNewNote({
        parentNoteId: todayNote.noteId,
        title: title,
        content: '',
        type: 'render'
    })).note;
    await renderNote.setLabel('clipType', 'clipper-HTML');
    await renderNote.setLabel('pageUrl', url);
    await renderNote.setLabel('pageTitle', title);

    // create child `content.html`
    var wrapped_content = template.replace("%%CONTENT%%", encodeURIComponent(content));
    const htmlNote = (await api.createNewNote({
        parentNoteId: renderNote.noteId,
        title: 'content.html',
        content: wrapped_content,
        type: 'file',
        mime: 'text/html'
    })).note;
    await htmlNote.setLabel('archived');

    // link renderNote to htmlNote
    await renderNote.setRelation('renderNote', htmlNote.noteId);

    res.status(200).send("{\"noteId\": \""+renderNote.noteId+"\"}");
}
else {
    res.send(500); 
}
```
Based on https://github.com/nil0x42/trilium-utils/blob/master/singlefile2trilium/singlefile2trilium-handler.js and https://github.com/html-screen-capture-js/html-screen-capture-js
