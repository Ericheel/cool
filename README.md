# cool

##### 记录

    private void recursion(Page page) {
        LOGGER.debug("Replacing page " + (page == null ? "null" : page.getTitle()));
        if (null != page) {
            if (page.hasChildren()) {
                List<Page> children = page.getChildren();
                for (Page childrenPage : children) {
                    recursion(childrenPage);
                }
            }
            String bodyAsString = page.getBodyAsString();
            if ((null != bodyAsString) && (!"".equals(bodyAsString))) {
                Pattern pattern = Pattern.compile(destination);
                Matcher matcher = pattern.matcher(bodyAsString);
                StringBuffer stringBuffer = new StringBuffer();
                int start = 0;
                int count = 0;
                while (matcher.find()) {
                    int end = bodyAsString.indexOf(matcher.group(), matcher.end()) != -1 ? bodyAsString.indexOf(matcher.group(), matcher.end()) : bodyAsString.length();
                    String substring = bodyAsString.substring(start, end);
                    if (substring.matches(".*?(<[^>!]*" + matcher.group() + ".*>).*$")) {
                        matcher.appendReplacement(stringBuffer, matcher.group());
                    } else {
                        matcher.appendReplacement(stringBuffer, replaceText);
                        pageAlreadyReplacedMap.put(page.getTitle(), ++count);
                    }
                    start = matcher.end();
                }
                matcher.appendTail(stringBuffer);
                String newBodyAsString = stringBuffer.toString();
                if (count > 0) {
                    try {
                        Page clonePage = (Page) page.clone();
                        page.setBodyAsString(newBodyAsString);
                        pageManager.saveContentEntity(page, clonePage, null);
                    } catch (CloneNotSupportedException e) {
                        LOGGER.error("Error in cloning page " + page.getTitle(), e);
                    }
                }
            }
            String originPageTitle = page.getTitle();
            String replacedTitle = originPageTitle.replace(destination, replaceText);
            if ("".equals(replaceText.trim())) {
                if (destination.equals(originPageTitle)) {
                    titleFailedMap.put(originPageTitle, replaceText);
                    LOGGER.debug(originPageTitle + " will be deleted or set as blank,this operation is not allowed");
                }
            } else {
                try {
                    if (!originPageTitle.equals(replacedTitle)) {
                        pageManager.renamePage(page, replacedTitle);
                        titleCount++;
                    }
                } catch (DuplicateDataRuntimeException e) {
                    titleFailedMap.put(originPageTitle, replacedTitle);
                    LOGGER.error("Page title duplicated,original page title is " + originPageTitle + " wanting title is " + replacedTitle, e);
                }
            }
        }

    }
