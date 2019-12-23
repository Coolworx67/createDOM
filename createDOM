function createDOM(domObj) {
  /*	
    // 2017.10.25 - added stringToObject - allows for camelcase/hyphen quick strings for css properties in objects
    // 2017.10.26 - refactored jsToCSS to check cssIntegerProps before adding 'px' to numbers
    // 2017.12.13 - added scopedCSS back.
    // 2017.12.22 - fixed scopedCSS to work with @media and @keyframes
    // 2018.03.01 - fixed scopedCSS to automatically add wrapper <div>
    // 2019.05.18 - added letterSpacing to cssPixalProps
   */

  let
    dom = {},
    eventTypes = ['click', 'dblclick', 'focus', 'blur', 'select',
      'submit', 'reset', 'keydown', 'keypress', 'keyup', 'change',
      'mouseover', 'mouseout', 'mousedown', 'mouseup', 'mouseenter',
      'mouseleave', 'mousewheel'
    ],

    cssPseudoClasses = ['hover', 'focus', 'active'],

    cssPseudoElements = ['after', 'before', 'firstLetter',
      'firstLine', 'selection'
    ],

    cssPixelProps = ['width', 'height', 'fontSize', 'borderRadius', 'padding',
      'paddingLeft', 'paddingTop', 'paddingRight', 'paddingBottom', 'margin',
      'marginLeft', 'marginRight', 'marginTop', 'marginBottom', 'top', 'left',
      'right', 'bottom', 'gridGap', 'letterSpacing'
    ];

  dom.delimiter = domObj && domObj.delimiter || '|';

  if (Array.isArray(domObj)) {
    dom = {
      tag: 'FRAGMENT',
      delimiter: domObj.delimiter || '|',
      desc: domObj
    };
  } else if (typeof domObj === 'string') {
    dom = Object.assign(dom, parseTag(domObj.replace(/\s\s+/g, ' ')));
  } else if (typeof domObj === 'object') {
    dom = Object.assign(dom, domObj);
  }

  if (dom.className) {
    if (Array.isArray(dom.className)) {
      dom.className = dom.className.join(' ');
    }
    dom.className.replace(/,/g, ' ').replace(/\s\s+/g, ' ');
  }

  if (dom.scopedCSS) {
    let scss = dom.scopedCSS;
    delete dom.scopedCSS;
    dom = {
      scopedCSS: scss,
      desc: dom
    };

    const scopedClass = '_' + makeID(2);
    dom.className = dom.className ? dom.className + ' ' + scopedClass : scopedClass;

    dom.css = Object.keys(dom.scopedCSS).reduce((o, k) => {
      if (k.startsWith('@keyframes')) {
        o[k] = dom.scopedCSS[k];
        return o;

      } 
      else if (k.startsWith('@media')) {
        const subCSS = Object.keys(dom.scopedCSS[k]).reduce((_o, _k) => {
          let _key = `.${scopedClass} ${_k}`;
          _o[_key] = dom.scopedCSS[k][_k];
          return _o;
        }, {});

        o[k] = subCSS;
        return o;

      } 
      else {
        let key = `.${scopedClass} ${k}`;
        o[key] = dom.scopedCSS[k];
        return o;

      }
    }, {});


    delete dom.scopedCSS;
  }

  if (dom.tag) {
    let parsedTag = parseTag(dom.tag);
    if (parsedTag.className && dom.className) {
      parsedTag.className += ' ' + dom.className;
    }
    dom = Object.assign(dom, parsedTag);
  } else dom.tag = 'DIV';

  if (dom.css) {
    writeCSS(dom.css);
    delete dom.css;
  }

  let
    cleanTag = dom.tag.trim().toUpperCase(),
    elm = (cleanTag === 'FRAGMENT') ?
    document.createDocumentFragment() : document.createElement(cleanTag);

  eventTypes.forEach(function(etype) {
    if (dom[etype]) {
      if (!dom.events) dom.events = {};
      dom.events[etype] = dom[etype];
      delete dom[etype];
    }
  });

  if (dom.html || dom.desc || dom.text) {
    let appendCalls = {
      html: function() {
        elm.innerHTML = elm.innerHTML + dom.html;
      },
      text: function() {
        elm.appendChild(document.createTextNode(dom.text));
      },
      desc: function() {
        if (Array.isArray(dom.desc)) dom.desc.forEach(function(i) {
          appendElm(i);
        });
        else appendElm(dom.desc);
      }
    };

    Object.keys(dom).forEach(function(k) {
      if (appendCalls.hasOwnProperty(k)) {
        appendCalls[k]();
        delete dom[k];
      }
    });
  }

  if (dom.events) {
    Object.keys(dom.events).forEach(function(evt) {
      if (typeof window.addEventListener === 'function')
        elm.addEventListener(evt, dom.events[evt], false);
      else if (typeof window.attachEvent === 'function')
        elm.attachEvent('on' + evt, dom.events[evt]);
      else elm['on' + evt] = dom.events[evt];
      if (dom.registerEvents) {
        dom.registerEvents(elm, evt, dom.events[evt]);
      }
    });
    delete dom.events;
    delete dom.registerEvents;
  }

  delete dom.delimiter;
  delete dom.tag;

  if (dom.style && typeof dom.style === 'object') {
    dom.style = jsToCSS(dom.style);
  }

  Object.keys(dom).forEach(function(k) {
    let prop = checkReplaceHash(k);
    setAttr(prop, dom[k]);
  });

  return elm;

  // Sub Functions

  function makeID(num) {
    num = num || 1;
    let res = '';
    while (!!num) {
      let array = new Uint32Array(num);
      window.crypto.getRandomValues(array);
      for (let i = 0; i < array.length; i++) {
        res += array[i].toString(36);
      }
      num--;
    }
    return res;
  }

  function isNumber(val) {
    return (typeof val === 'number' && !isNaN(val));
  }

  function isElm(el) {
    return (el.nodeType && el.nodeType === 1);
  }

  function stringToObject(str, delim = ';', kvpDelim = ':') {
    if (!str) return {};
    if (typeof str === 'object') return str;
    let errorString = `Error: stringToObject.\n"${str}"\n is not properly formatted`;

    let arr = str.trim().split(delim);

    let obj = arr.reduce((o, kvp) => {
      if (kvp.trim()) {
        try {
          let pair = kvp.split(kvpDelim);
          o[pair[0].trim()] = isNaN(pair[1].trim()) ?
            pair[1].trim() : +pair[1].trim();
        } catch (err) {
          throw (errorString);
        }
      }
      return o;
    }, {});
    return obj;
  }

  function appendElm(el) {
    if (isElm(el)) elm.appendChild(el);
    else {
      let childElm = (typeof el === 'object') ?
        Object.assign({}, el) : (typeof el === 'string') ?
        Object.assign({}, parseTag(el.replace(/\s\s+/g, ' '))) : {};

      childElm.delimiter = childElm.delimiter || dom.delimiter;
      elm.appendChild(createDOM(childElm));
    }
  }

  function setAttr(key, val) {
    try {
      if (key === 'class') {
        val.split(' ').forEach(function(c) {
          elm.classList.add(c);
        });
      } else elm.setAttribute(key, val);
    } catch (err) {
      console.log(err);
    }
  }

  function camelToHyphen(s) {
    let res = s.replace(/[a-z][A-Z]/g, function(str, offset) {
      return str[0] + '-' + str[1].toLowerCase();
    });
    return res;
  }

  function jsToCSS(o) {
    let _o = typeof o === 'string' ?
      stringToObject(o) : Object.assign({}, o);
    let res = '';
    Object.keys(_o).forEach(function(k) {
      if (isNumber(_o[k]) && cssPixelProps.includes(k)) {
        _o[k] += 'px';
      }
      res = res.concat(camelToHyphen(k) + ':' + _o[k] + ';');
    });
    return res;
  }

  function parseTag(str) {
    let
      attrs = str.split(dom.delimiter),
      tagString = attrs.shift().trim(),
      res = {},
      symHash = {
        '.': 'className',
        '#': 'id',
        '~': 'name',
        '^': 'type'
      },
      flags = Object.keys(symHash),
      prop = 'tag',
      classStart = false;

    for (let ii = 0; ii < tagString.length; ii++) {
      if (!flags.includes(tagString[ii])) {
        if (prop === 'className' && classStart) {
          res.className = res.className ? res.className + ' ' : '';
          classStart = false;
        }
        res[prop] = res[prop] ? res[prop] + tagString[ii] : tagString[ii];
        continue;
      } else {
        prop = symHash[tagString[ii]];
        if (prop === 'className') classStart = true;
      }
    }

    if (attrs.length > 0) {
      attrs.forEach(function(kp) {
        let p = kp.split('=');
        if (p[0].trim() !== '') {
          res[p[0].trim()] = (p[1] && p[1].trim) ? p[1].trim() : '';
        }
      });
    }
    res.tag = res.tag || 'DIV';
    return res;
  }

  function createStyleSheet() {
    if (document.getElementById('createDOM_Generated_Style_Sheet')) {
      return document.getElementById('createDOM_Generated_Style_Sheet');
    } else {
      let style = createDOM({
        tag: 'style',
        type: 'text/css',
        media: 'screen',
        id: 'createDOM_Generated_Style_Sheet'
      });

      document.getElementsByTagName('head')[0].appendChild(style);
      return style;
    }
  }

  function writeStyleSheet(o) {
    let style = createStyleSheet();

    if (!(style.sheet || {}).insertRule) {
      (style.styleSheet || style.sheet).addRule(o.name, o.rules);
    } else {
      style.appendChild(document.createTextNode(o.name + " {" + o.rules + "}"));
    }
  }

  function writeKeyframes(keyframeObj, name) {
    let style = createStyleSheet();
    let str = name + '{';

    Object.keys(keyframeObj).forEach(function(frame) {
      str += frame + '{';
      str += jsToCSS(keyframeObj[frame]) + '}';
    });

    str += '}';
    style.appendChild(document.createTextNode(str));
  }

  function writeMediaQuery(css, mediaObj) {
    let style = createStyleSheet();
    let str = mediaObj + '{';

    Object.keys(css[mediaObj]).forEach(function(mediaRuleObj) {
      let mRules = css[mediaObj][mediaRuleObj];

      Object.keys(mRules).forEach(function(mRule) {

        if (cssPseudoClasses.includes(mRule) || cssPseudoElements.includes(mRule)) {
          let colons = cssPseudoElements.includes(mRule) ? '::' : ':';
          str += mediaRuleObj + colons + mRule + ' ' + '{';
          str += jsToCSS(css[mediaObj][mediaRuleObj][mRule]) + '}';
          delete css[mediaObj][mediaRuleObj][mRule];
        }

      });

      str += mediaRuleObj + ' ' + '{';
      str += jsToCSS(css[mediaObj][mediaRuleObj]) + '}';
    });

    str += '}';
    style.appendChild(document.createTextNode(str));

  }

  function writeCSS(css) {
    Object.keys(css).forEach(function(item) {
      if (item.substr(0, '@keyframes'.length) === '@keyframes') {
        writeKeyframes(css[item], item);
        return;
      }

      if (item.substr(0, '@media'.length) === '@media') {
        writeMediaQuery(css, item);
        return;
      }

      Object.keys(css[item]).forEach(function(prop) {
        if (cssPseudoClasses.includes(prop) || cssPseudoElements.includes(prop)) {
          let colons = cssPseudoElements.includes(prop) ? '::' : ':';

          if (prop === 'selection') {
            writeStyleSheet({
              name: item + colons + '-moz-' + camelToHyphen(prop),
              rules: jsToCSS(css[item][prop])
            });
          }

          writeStyleSheet({
            name: item + colons + camelToHyphen(prop),
            rules: jsToCSS(css[item][prop])
          });
          delete css[item][prop];
          return;
        }
      });

      writeStyleSheet({
        name: item.replace(/\$/g, ':'),
        rules: jsToCSS(css[item])
      });
      return;
    });
  }

  function checkReplaceHash(prop) {
    let replaceHash = {
      'className': 'class',
      'data_': 'data-',
      '@': 'v-on:',
      'v_on_': 'v-on:',
      'v_bind_': 'v-bind:',
      'v_if': 'v-if',
      'v_for': 'v-for',
      'v_text': 'v-text',
      'v_html': 'v-html',
      'v_else_if': 'v-else-if',
      'v_else': 'v-else',
      'v_show': 'v-show',
      'v_pre': 'v-pre',
      'v_model': 'v-model',
      'v_once': 'v-once',
      'v_cloak': 'v-cloak'
    };

    let res = prop;

    Object.keys(replaceHash).forEach(function(k) {
      if (prop.startsWith(k)) {
        res = prop.replace(k, replaceHash[k]);
        if (res.startsWith('v-on:') && res.includes('$')) {
          res = res.replace(/\$/g, '.');
        }
        return;
      }
    });

    return res;
  }
}
