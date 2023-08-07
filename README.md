# TYPO3 v12.4 - Constants Data Processor
Generate an array from a constants csv string that will be decoded again and assigned to FLUIDTEMPLATE as variable. Derived from the existing TYPO3 data processors: https://github.com/TYPO3/typo3/tree/main/typo3/sysext/frontend/Classes/DataProcessing 
## Data Processor Class
my_site_package/Classes/DataProcessing/SelectedPagesProcessor.php
```
<?php

declare(strict_types=1);

/*
 * This file is part of the TYPO3 CMS project.
 *
 * It is free software; you can redistribute it and/or modify it under
 * the terms of the GNU General Public License, either version 2
 * of the License, or any later version.
 *
 * For the full copyright and license information, please read the
 * LICENSE.txt file that was distributed with this source code.
 *
 * The TYPO3 project - inspiring people to share!
 */

namespace Vendor\SitePackage\DataProcessing;

use TYPO3\CMS\Core\Utility\GeneralUtility;
use TYPO3\CMS\Frontend\ContentObject\ContentObjectRenderer;
use TYPO3\CMS\Frontend\ContentObject\DataProcessorInterface;

/**
 * Class for data processing comma separated values string from constants
 * This processor generates an array from a csv string that will be
 * decoded again and assigned to FLUIDTEMPLATE as variable.
 *
 * Options:
 * if         - TypoScript if condition
 * pagesvalue - A list of page id's (e.g. 0,1,2)
 * as         - The variable to be used within the result
 *
 * Example TypoScript configuration:
 * 10 = Vendor\SitePackage\DataProcessing\SelectedPagesProcessor
 * 10 {
 *   pagesvalue = {$page.theme.selectedpages.selectedpagesValue}
 *   as = selectedpages
 * }
 */
class SelectedPagesProcessor implements DataProcessorInterface
{
    /**
     * @param ContentObjectRenderer $cObj The data of the content element or page
     * @param array $contentObjectConfiguration The configuration of Content Object
     * @param array $processorConfiguration The configuration of this processor
     * @param array $processedData Key/value store of processed data (e.g. to be passed to a Fluid View)
     * @return array the processed data as key/value store
     */
    public function process(
        ContentObjectRenderer $cObj,
        array $contentObjectConfiguration,
        array $processorConfiguration,
        array $processedData
    ) {
        if (isset($processorConfiguration['if.']) && !$cObj->checkIf($processorConfiguration['if.'])) {
            return $processedData;
        }
        // pages by comma separated list
        $pagesIdList = $cObj->stdWrapValue('pagesvalue', $processorConfiguration ?? []);
        $pages = [];
        if ($pagesIdList) {
            $pages = GeneralUtility::intExplode(',', (string)$pagesIdList, true);
            // set the pages into a variable, default "pages"
            $targetVariableName = $cObj->stdWrapValue('as', $processorConfiguration, 'pages');
            $processedData[$targetVariableName] = $pages;
        }
        return $processedData;
    }
}
```
## FLUIDTEMPLATE with SelectedPagesProcessor
my_site_package/Configuration/TypoScript/setup.typoscript
```
page = PAGE
page {
    10 = FLUIDTEMPLATE
    10 {
        dataProcessing {    
            10 = Vendor\SitePackage\DataProcessing\SelectedPagesProcessor
            10 {
                pagesvalue = {$page.theme.selectedpages.selectedpagesValue}
                as = selectedpages
            	}        
            }
        }        
    }        
```        
## Constants - Define the input field for the Constant Editor
my_site_package/Configuration/TypoScript/constants.typoscript
```
page {
    theme {        
        selectedpages {
            # cat=site package/selectedpages/125/; type=string; label=LLL:EXT:site_package/Resources/Private/Language/locallang_be.xlf:constants.selectedpages_uid
            selectedpagesValue =
            }
    	}
    }
```
## Example Template - Use the variable to display a partial only on certain pages (data.uid)
my_site_package/Resources/Templates/Page/Default.htm
```
<f:for each="{selectedpages}" as="selectedpagesItem">
    <f:if condition="{data.uid} == {selectedpagesItem}">
        <div class="container">
            <f:render partial="somepartial" arguments="{_all}"/>
        </div>
    </f:if>
</f:for>
``` 
